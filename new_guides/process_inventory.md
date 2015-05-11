## Inventory
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
* All locations from where inventory is shipped
* Each `StockLocation` has many `stock_items` and `stock_movements`
* Created in the admin interface via (Configuration → Stock Locations).
* A `StockItem` is added to newly created `StockLocation` for every variant in an application

### Stock Items
* Represent inventory at a stock location for a specific variant
* `stock_items` are created automatically for each `StockLocation`
* Stock item count on hand can be increased or decreased by creating stock movements

### Stock Movements
* Allows management of inventory of a stock item for a `StockLocation`
* Created in the admin interface via (Product XYZ → Stock Management) where you can increase or
decrease count on hand for the variant at a stock location

### Stock Transfers
* Allows movement of inventory in bulk from one stock location to another
* Done in the admin interface via (Configuration → Stock Transfers) by selecting a source location,
a destination location, one or more variants and quantity for each variant individually if needed

## Return Authorizations
After shipment, admins can approve return of an order of a part of it via the "Return
Authorizations" tab in the single order console. To create a new return authorization, you should
indicate which part of the order is being returned, what the reason for the return is, and what the
resulting credit should be. The sale price of the product is shown for reference, but you can choose
any value you like.
After authorization is created, you can return to its edit page and click 'Received' to confirm the
return. This creates a credit adjustment on the order, which you can apply (i.e. refund) to the
order's credit card via the payments screen. Spree will log the events in the order's history.
