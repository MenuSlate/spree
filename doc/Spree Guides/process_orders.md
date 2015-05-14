# Orders Guide

## Components
### Order (Model)
See [here](/doc/Spree/Order.md)

### Line Item (Model)
See [here](/doc/Spree/LineItem.md)

### Adjustment (Model)
See [here](/doc/Spree/Adjustment.md)

### Addresses
An order can link to two `Address` objects:
1. The shipping address indicates where the order's product(s) should be shipped to determines 
shipping methods available
2. The billing address indicates where the user paying for the order is and can alter the tax 
rate which can change the final order total

## Updating an Order
If you change an `Order` object in code in anyway and you want to update totals and associated
adjustments and shipments, call `update!` method on the object (this will call `OrderUpdater`)

For example, if you create or modify an existing payment for the order which would change the order's
`payment_state` to a different value, calling `update!` will cause the `payment_state` to be
recalculated for that order.

Another example is if a `LineItem` within the order had its price changed. Calling `update!` will
cause the totals for the order to be updated, the adjustments for the order to be recalculated, and
then a final total to be established.

## Order Manual Entry
* Can be done through the Admin Interface (Orders -> New Order)
* Visually detailed in [Spree User Guide]
(https://guides.spreecommerce.com/user/entering_orders.html)

## Order Returns
See it under [Inventory Guide](/doc/Spree Guides/process_inventory.md#Returns)
