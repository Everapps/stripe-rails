## 0.2.6 (2013-07-12)
* add config.stripe.api_version setting to support api versioning

## 0.2.5 (2013-03-18)
* make the default max redemptions 1
* add stripe::coupons::reset! task to redefine all coupons

## 0.2.2 (2013-01-09)
* bugfix allowing creation of coupons without max_redemptions

## 0.2.1 (2012-12-17)
* manage coupons with config/stripe/coupons.rb

## 0.2.0 (2012-12-13)

* out of the box support for webhooks and critical/non-critical event handlers
* add :only guards for which webhooks you respond to-
* move stripe.js out of asset pipeline, and insert it with utility functions

## 0.1.0 (2012-11-14)

* add config/stripe/plans.rb to define and create plans
* use `STRIPE_API_KEY` as default value of `config.stripe.api_key`
* require stripe.js from asset pipeline
* autoconfigure stripe.js with config.stripe.publishable_key.
* add rake stripe:verify to ensure stripe.com authentication is configured properly

## 0.0.1 (2012-10-12)

* basic railtie
