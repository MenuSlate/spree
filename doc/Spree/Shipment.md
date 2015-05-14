## Shipment (Model)
* `Shipment` object is created during checkout to track shipping method the order it applies to

### Attributes
* `number`: unique identifier, begins with "H" and ends in an 11-digit number. Can be used to find
the order by calling `Shipment.find_by(number: number)`.=
* `tracking`: identifier for shipping provider (i.e. FedEx, UPS, etc)
* `shipped_at`
* `state`
* `stock_location_id`: ID of Stock Location where shipment will be sourced from

### States
* `pending`: shipment has backordered inventory units and or order isn't paid
* `ready`: shipment has no backordered inventory units and order is paid
* `shipped`
* `canceled`: All order shipments are cancelled and products will be restocked. Shipment resumes
only if order is "resumed"

> For more info see [Shipping Guide](process_shipping.md)
