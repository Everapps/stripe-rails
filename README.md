# Stripe::Rails: A Rails Engine for use with [stripe.com](https://stripe.com)

This gem can help your rails application integrate with Stripe in the following ways

* manage stripe configurations in a single place.
* makes stripe.js available from the asset pipeline.
* manage plans and coupons from within your app.
* painlessly receive and validate webhooks from stripe.

## Installation

Add this line to your application's Gemfile:

    gem 'stripe-rails'

If you are going to be using [stripe.js][1] to securely collect credit card information
on the client, then add the following to app/assets/javascripts/application.js

    //= require stripe

### Setup your API keys.

You will need to configure your application to authenticate with stripe.com
using [your api key][1]. There are two methods to do this, you can either set the environment
variable `STRIPE_API_KEY`, or use the rails configuration setting `config.stripe.api_key`.
In either case, it is recommended that you *not* check in this value into source control.

You can verify that your api is set up and functioning properly by running the following command:

    rake stripe:verify

If you are going to be using stripe.js, then you will also need to set the value of your
publishiable key. A nice way to do it is to set your test publishable for all environments:

    # config/application.rb
    # ...
    config.stripe.publishable_key = 'pk_test_XXXYYYZZZ'

And then override it to use your live key in production only

    # config/environments/production.rb
    # ...
    config.stripe.publishable_key = 'pk_live_XXXYYYZZZ'

This key will be publicly visible on the internet, so it is ok to put in your source.

### Setup your payment configuration

If you're using subscriptions, then you'll need to set up your application's payment plans
and discounts. `Stripe::Rails` lets you automate the management of these definitions from
within the application itself. To get started:

    rails generate stripe:install

this will generate the configuration files containing your plan and coupon definitions:

      create  config/stripe/plans.rb
      create  config/stripe/coupons.rb

### Configuring your plans

Use the plan builder to define as many plans as you want in `config/stripe/plans.rb`

    Stripe.plan :silver do |plan|
      plan.name = 'ACME Silver'
      plan.amount = 699 # $6.99
      plan.interval = 'month'
    end

    Stripe.plan :gold do |plan|
      plan.name = 'ACME Gold'
      plan.amount = 999 # $9.99
      plan.interval = 'month'
    end

This will define constants for these plans in the Stripe::Plans module so that you
can refer to them by reference as opposed to an id string.

    Stripe::Plans::SILVER # => 'silver: ACME Silver'
    Stripe::Plans::GOLD # => 'gold: ACME Gold'

To upload these plans onto stripe.com, run:

    rake stripe:prepare

This will create any plans that do not currently exist, and treat as a NOOP any
plans that do, so you can run this command safely as many times as you wish.

NOTE: You must destroy plans manually from your stripe dashboard.

## Webhooks

Stripe::Rails automatically sets up your application to receive webhooks from stripe.com whenever
an payment event is generated. To enable this, you will need to configure your [stripe webooks][3] to
point back to your application. By default, the webhook controller is mounted at '/stripe/events' so
you would want to enter in `http://myproductionapp.com/stripe/events` as your url for live mode,
and `http://mystagingapp.com/stripe/events` for your test mode.

If you want to mount the stripe engine somewhere else, you can do so by setting the `stripe.endpoint`
parameter. E.g.

    config.stripe.endpoint = '/payment/stripe-integration'

Your new webook URL would then be `http://myproductionapp/payment/strip-integration/events`

### Responding to webhooks

Once you have your webhook URL configured you can respond to a stripe webhook *anywhere* in your
application just by including the Stripe::Callbacks module into your class and declaring a
callback with one of the callback methods. For example, to update a customer's payment status:

    class User < ActiveRecord::Base
      include Stripe::Callbacks

      after_customer_updated! do |event|
        customer = event.data.object
        user = User.find_by_stripe_customer_id(event.customer.id)
        if event.customer.delinquent
          user.is_account_current = false
          user.save!
        end
      end
    end

or to send an email with one of your customer's monthly invoices

    class InvoiceMailer < ActionMailer::Base
      include Stripe::Callbacks

      after_invoice_created! do |event|
        invoice = event.data.object
        user = User.find_by_stripe_customer(event.invoice.customer)
        new_invoice(user, event.invoice).deliver
      end

      def new_invoice(user, invoice)
        @user = user
        @invoice = invoice
        mail :to => user.email, :subject => '[Acme.com] Your new invoice'
      end
    end

The naming convention for the callback events is after__{callback_name}! where `callback_name`
is name of the stripe event with all `.` characters substituted with underscores. So, for
example, the stripe event `customer.discount.created` can be hooked by `after_customer_discount_created!`
and so on...

Each web hook is passed an instance of [`Stripe::Event`][4] corresponding to the event being
raised. By default, it is re-fetched securely from stripe.com to prevent damage to your system by
a malicious system spoofing real stripe events.

### Critical and non-critical hooks

So far, the examples have all used critical hooks, but in fact, each callback method comes in two flavors: "critical",
specified with a trailing `!` character, and "non-critical", which has no "bang" character at all. What
distinguishes one from the other is that _if an exception is raised in a critical callback, it will cause the entire webhook to fail_.

This will indicate to stripe.com that you did not receive the webhook at all, and that it should retry it again later until it
receives a successful response. On the other hand, there are some tasks that are more tangential to the payment work flow and aren't
such a big deal if they get dropped on the floor. For example, A non-critical hook can be used to do things like have a bot
notify your company's chatroom that something a credit card was successfully charged:

    class AcmeBot
      include Stripe::Callbacks

      def after_charge_succeeded do |event|
        announce "Attention all Dudes and Dudettes. Ya'll are so PAID!!!"
      end
    end

Chances are that if you experience a momentary failure in connectivity to your chatroom, you don't want the whole payment notification to fail.

See the [complete listing of all stripe events][5], and the [webhook tutorial][6] for more great information on this subject.

## Thanks

<a href="http://thefrontside.net">![The Frontside](http://github.com/cowboyd/therubyracer/raw/master/thefrontside.png)</a>

`Stripe::Rails` was developed fondly by your friends at [The FrontSide][7]. They are available for your custom software development
needs, including integration with stripe.com.

[1]: https://stripe.com/docs/stripe.js
[2]: https://manage.stripe.com/#account/apikeys
[3]: https://manage.stripe.com/#account/webhooks
[4]: https://stripe.com/docs/api?lang=ruby#events
[5]: https://stripe.com/docs/api?lang=ruby#event_types
[6]: https://stripe.com/docs/webhooks
[7]: http://thefrontside.net