## Inventory Guide
> Note: If you don't need to track inventory set `Spree::Config[:track_inventory_levels]` to `false`

* On-hand inventory is stored as a count of a variant's `StockItem`
* Back-ordered, sold, or shipped products are stored as individual `InventoryUnit` objects to have
relevant information attached to them
* New products created can be given a starting "on hand" inventory level. Setting a new level
subsequently will be handled appropriately, e.g. adding new on-hand inventory to an
out-of-stock product that has some backorders will first fill the backorders then update the
product with the remaining inventory count.

## Stock Management
### Stock Locations
See [here](/doc/Spree/StockLocation.md)

### Stock Items
See [here](/doc/Spree/StockItem.md)

### Stock Movements
See [here](/doc/Spree/StockMovement.md)

### Stock Transfers
See [here](/doc/Spree/StockTransfer.md)

## Returns
* Site administrators can issue RMAs (Return Merchandise Authorizations) and process returns
* RMAs can only be created for orders that have shipped
* Returns can either be
  * Reimbursement: This refunds a user through their original payment method when the items are 
  marked returned and approved
  * Exchange: Creates a new shipment to ship an exchange item to the user
* RMA value is based on sale price of the item(s) and an admin confirms the amount when 
reimbursement is issued (to adjust for handling fees, restocking fees, damages, etc.)
* A [Stock Location](stock_locations) for where the RMA item(s) is coming back has to be selected
* Once a package returns, an admin creates a "Customer Return" that lists authorized return item(s) 
received and to which [Stock Location](stock_locations)
* Once returned items are marked accepted a reimbursement can be created after which a refund is 
issued to the original credit card

## Returns (UI)
* After shipment, admins can approve return of an order or a part of it 
* To create a new return authorization, indicate
  * which part of the order is being returned
  * reason for the return 
  * resulting credit (sale price of the product is shown for reference)
* once items return go back to the return authorization and click 'Received'. This creates a 
credit adjustment on the order, which you can apply (i.e. refund) to the order's credit card via 
the payments screen
* Spree logs these events in the order's history
