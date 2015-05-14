## Adjustment Model
Tracks an adjustment, positive(`Shipping` or `Taxes`) or negative (`Promotion`) to the price of an 
*adjustable* which is either:
- [Order](orders)
- Order's [Line Item](orders#line-items)
- Order's [Shipments](shipments)

### Attributes:
* `amount` money amount of adjustment
* `label`: adjustment *source* (either TaxRate or PromotionAction) and other info
* `eligible`: whether the adjustment will count with this specific order
* `mandatory` If true the charge will NOT be removed from the order, even if it's zero and a record
will be created for inventory, expense and tax purposes
* `included`: Whether this adjustment affects offering final price and thus its tax adjustments
* `state`: either `open`, `closed` to updating automatically, or `finalized` and not changeable

## Adjustment Scopes
* `open`: All open adjustments
* `eligible`: All eligible adjustments for the order. Useful for determining which adjustments 
are applying to the adjustable
* `tax`: All adjustments which have a source that is a `Spree::TaxRate` object
* `price`: All adjustments which adjust a `Spree::LineItem` object.
* `shipping`: All adjustments which adjust a `Spree::Shipment` object.
* `promotion`: All adjustments where the source is a `Spree::PromotionAction` object.
* `optional`: All adjustments which are not `mandatory`.
* `return_authorization`: All adjustments where the source is a `Spree::ReturnAuthorization`.
* `eligible`: Adjustments which have been determined to be `eligible` for their adjustable.
* `charge`: Adjustments which *increase* the price of their adjustable.
* `credit`: Adjustments which *decrease* the price of their adjustable.
* `optional`: Adjustments which are not mandatory.
* `included`: Adjustments calculated/included in the object's price. Typically item tax adjustments.
* `additional`: Adjustments calculated on top of the object's price. The default. e.g. sales taxes

There are helper methods to return the different types of adjustments:
scope :shipping, -> { where(adjustable_type: 'Spree::Shipment') }
These scopes can be called on either `Adjustment` class itself, or on an `adjustments` association
Spree::Adjustment.eligible
order.adjustments.eligible
line_item.adjustments.eligible
shipment.adjustments.eligible

## Adjustment Associations
order.adjustments # retrieve adjustments on the order itself
order.all_adjustments # retrieve all adjustments for all line items, shipments and the order itself
order.line_item_adjustments # retrieve line item adjustments
order.shipment_adjustments # retrieve adjustments applied to shipments

> Closed adjustments can be re-opened and changed, up to the moment when the order is shipped. At
that point, the adjustment is finalized and cannot be changed
