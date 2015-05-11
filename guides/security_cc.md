# Credit Card Security

* In production mode, credit card data is transmitted only once via SSL
* Data is immediately relayed to your payment gateway and then discarded
* Only last four digits of the credit card and the expiration month and date are stored
* Other data is never stored in the database, not even temporarily. It exists in memory on the 
server for a fraction of a second before it's discarded

### Payment Profiles
* Service provided by some payment gateways
* Default checkout process in Spree assumes a gateway that allows for third party support for 
payment profiles
* Prior to profiles, the only safe way was to have card info entered on the final step with no
"review and confirm" step for the user

#### Process
* Credit card gateway is stores the credit card information
* Your application stores only a token that can be used to authorize future charges on that same 
card without having to actually store it

#### Benefits
* Secure and PCI compliant means of storing the users credit card information 
* Merchants to issue refunds to the credit card
* Merchants to make changes to an existing order without having to leave Spree and use the 
gateway provider's website
* A "confirmation" step before an order is processed since card number is stored 
securely on the payment step and can be used to perform standard authorization/capture via
the secure token provided by the gateway

#### Customizing
* All `Gateway` classes have a `payment_profiles_supported?` method which indicates whether or 
not payment profiles are supported
* If you are adding Spree support to a `Gateway` you should also implement the `create_profile` 
method. The following is an example:
```

# Create a new CIM customer profile ready to accept a payment
def create_profile(payment)
  if payment.source.gateway_customer_profile_id.nil?
    profile_hash = create_customer_profile(payment)
    payment.source.update_attributes({
      :gateway_customer_profile_id => profile_hash[:customer_profile_id],
      :gateway_payment_profile_id => profile_hash[:customer_payment_profile_id])
    })
  end
end
```
