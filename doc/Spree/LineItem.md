## Line Item  (Model)
* Keeps track of items within the context of an order and links orders and variants 
(products#variants)
* When a variant is added to an order, the price of that item is tracked along with the line item
 to preserve that data. If the variant's price were to change, then the line item would still 
 have a record of the price at the time of ordering.
* Inventory tracking notes 
$$$ Update this section after Chris+Brian have done their thing. $$$
