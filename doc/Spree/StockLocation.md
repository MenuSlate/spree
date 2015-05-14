## Stock Location (Model)
* Any location from where inventory is shipped
* Each `StockLocation` has many `stock_items` and `stock_movements`
* Created in the admin interface via (Configuration ? Stock Locations)
* A `StockItem` is added to newly created `StockLocation` for every variant in an application
