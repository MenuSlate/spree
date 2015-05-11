## Shipment
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

## Shipping Methods
* The Actual services used to send the product. For example:
    * UPS Ground
    * UPS One Day
    * FedEx 2Day
    * FedEx Overnight
    * DHL International
* Each method is only applicable to a specific `Zone` to prevent sending a package internationally
using domestic shipping method
* Shipping methods can have multiple shipping categories assigned
* Shipping categories of items in an order can determine the order's available shipping methods

## Zones
Order Shipping Address define the zone a customer is in and limits Shipping Methods available

## Shipping Categories
* Can be created in the admin interface ("Configuration" -> "Shipping Categories") and then
assigned to products ("Products" -> "Edit")
* Useful when shipping costs vary depending on product(s) selected. For example "Light", "Heavy"..
* If shipping cost for all products is the same, all products would be assigned one shipping category
* During checkout, shipping categories of products in the order will determine which calculator will
be used to price each Shipping Method

## Calculators
* Calculate shipping price for each available Shipping Method
* Spree ships with 5 default Calculators:
    * Flat rate (per order)
    * Flat rate (per item/product)
    * Flat percent
    * Flexible rate (flat rate for first product then a different flat rate for each additional)
    * Price sack

## Shipment Setup Examples
### Simple Setup
* Bruce Wayne Enterprises sells suits to US and Europe
* Bruce ships from a single location
* Bruce works with 2 carriers:
    * USPS Ground (to US): charges $5 for first suit and $2 for each additional
    * FedEx (to EU): charges $10 each

Spree Setup:
* 1 Shipping Category: Products are similar so a single category all products are assigned to suffice
* 1 Stock Location: Bruce ships all items from one location (cave) so he can use spree's default
* 2 Shipping Methods (Configuration->Shipping Methods) as follows:

| Name        | Zone   | Calculator             |
|-------------|--------|------------------------|
| USPS Ground | US     | Flexi Rate($5,$2)      |
| FedEx       | EU_VAT | FlatRate-per-item($10) |

### Advanced Setup
* Wayne Enterprises stopped EU operations as their suits were deemed "conspicuous religious symbols"
* Bruce now sells products to a single zone (US)
* Bruce ships from 2 locations (Stock Locations):
    * Gotham
    * Los Angeles
* Bruce products are classified into 3 Shipping Categories:
    * Light
    * Regular
    * Heavy
* Bruce works with 3 carriers now (Shipping Methods):
    * FedEx: Charges $10 for light, $2 for regular, $20 for first heavy item $15 for each additional
    * DHL: Charges $5 for light or regular and $50 for heavy
    * USPS: Charges $8 for light or regular and $20 for heavy

Spree Setup:
* 4 Shipping Categories: Default, Light, Regular and Heavy
* 3 Shipping Methods (Configuration->Shipping Methods): FedEx, DHL, USPS
* 2 Stock Locations (Configuration->Stock Locations): Gotham, Los Angeles

| S. Category/Method | DHL            | FedEx               | USPS           |
|--------------------|----------------|---------------------|----------------|
| Light              | Per Item ($5)  | Flat Rate ($10)     | Per Item ($8)  |
| Regular            | Per Item ($5)  | Per Item ($2)       | Per Item ($8)  |
| Heavy              | Per Item ($50) | Flexi Rate($20,$15) | Per Item ($20) |

## UI
* Admins update `Shipment` record with actual shipping cost and a tracking code and can also confirm
the dispatch (only once)
* Dispatch confirmation causes a shipping date to be set as the time of confirmation

## Advanced Shipping Methods
### Extensions
There are a few extensions which provide additional shipping methods, including support for fees
imposed by common carriers, bulk orders, etc.

## Product Configuration
* Order shipments contain standard information such as:
    * Shipping address
    * Variants and quantities
    * Weight and dimension info for a variant (if available)
* Product's `ShippingCategory` adds product-specific info for shipping calculations
* `ShippingCategory` is a wrapper of a string from which a calculator could extract shipping prices
e.g. "Fixed $20", "Fixed $40"..
* Admins can assign products to ShippingCategories or include additional info in variants that
enable the calculator to determine results

### Variant Configuration
Variants can have weight and dimension info that several shipping method calculators use if present

## Shipping Instructions
* `Config[:shipping_instructions]` controls collection of additional shipping instructions. This is
turned off (`false`) by default
* If an order has shipping instructions attached they'll be shown in the order's shipment admin page
* Instructions are currently attached to order and not to shipments

## The Active Shipping Extension
`spree_active_shipping` extension uses `active_shipping` gem to interface with carrier APIs such as
USPS, Fedex and UPS to provide Spree-compatible calculators for the different delivery services of
those carriers. More at https://github.com/spree/spree_active_shipping

## Filtering Shipping Methods On Criteria Other Than Zone
Ordinarily, zone of the shipping address determines available shipping methods:
```
class Spree::Stock::Estimator
  def shipping_methods(package)
    shipping_methods = package.shipping_methods
    shipping_methods.delete_if { |ship_method| !ship_method.calculator.available?(package.contents)}
    shipping_methods.delete_if { |ship_method| !ship_method.include?(order.ship_address) }
    shipping_methods.delete_if { |ship_method| !(ship_method.calculator.preferences[:currency].nil?
    || ship_method.calculator.preferences[:currency] == currency) }
    shipping_methods
  end
end
```
To disable a shipping method, calculator's `available?` method must be overridden:
```
class Calculator::Usps::FirstClassMailParcels < Calculator::Usps::Base
  def self.description
    "USPS First-Class Mail Parcel"
  end
  def available?(order)
    multiplier = 1.3
    weight = order.line_items.inject(0) do |weight, line_item|
      weight + (line_item.variant.weight ? (line_item.quantity
                                          * line_item.variant.weight
                                          * multiplier) : 0)
    end
    #if weight in ounces > 13, then First Class Mail is not available for the order
      weight > 13 ? false : true
  end
end
```

## Split Shipments
### Creating Proposed Shipments
1. Determining shipments for an order is triggered by calling `create_proposed_shipments` on an
`Order` object while transitioning to `delivery` state during checkout. This deletes any existing
shipments for the order and determines possible shipments available
2. `create_proposed_shipments` calls `Stock::Coordinator.new(@order).packages` which returns an
array of packages
3. A `Splitter` object determines which items belong in which package when they are being built
4. Available packages are converted to shipments on the order object
5. Shipping rates are determined and inventory units are created
6. Checkout process continue to delivery step

### Components of Split Shipments
#### (1) The Coordinator
* Starting point for determining shipments when calling `create_proposed_shipments` on an order
* Goes through each `StockLocation` available and determines what can be shipped from there
* `Stock::Coordinator` ultimately returns an array of packages thats converted into shipments for an
 order by calling `to_shipment` on them

#### (2) The Packer
* `Stock::Packer` Determines possible packages for a given `StockLocation` and order
* Uses rules in `Splitters` classes to determine what packages can be shipped from a `StockLocation`
* Splitters are run in sequence so each Splitter takes the output of the one that came before it

##### Example
* Stock location has two splitters
    * Splitter 1: Any order over 50lbs should be shipped in separate package from items weighing less
    * Splitter 2: catch-all for any item less than 50lbs
* Order has one item weighting 60lbs and two items weighing less
* Packer would use splitters' rules to come up with two separate packages:
    * Package 1: will contain the single 60lb item
    * Package 2: will contain the other two items

##### Default Splitters
###### Shipping Category Splitter
Runs first and splits an order into packages based on the shipping categories so each package will
only have products of the same shipping category
###### Weight Splitter
* Splits an order into packages based on a weight threshold so each package has a mass weight.
* If a new item is added to the order and it causes a package to go over the weight threshold, a new
package will be created so that all packages weigh less than the threshold.
* Weight threshold defaults to `150` and is changed through `Stock::Splitter::Weight.threshold`  in
an initializer

##### Custom Splitters
1. Inherit from `Stock::Splitter::Base`. For an example of a simple splitter, take a look at
[weight based splitter] which pulls items with a weight greater than 150 into their own shipment
2. Add the following to your application's spree initializer:
```
Rails.application.config.spree.stock_splitters << Spree::Stock::Splitter::CustomSplitter
```
3. You can completely override the splitters in Spree, rearrange them, etc.
To do this, add the following to your `spree.rb` file:
```
Rails.application.config.spree.stock_splitters = [
  Spree::Stock::Splitter::CustomSplitter,
  Spree::Stock::Splitter::ShippingCategory
]
```
4. Or if you don't want to split packages just set the option above to an empty
array. e.g. a store with the following configuration in spree.rb won't have any
package splitted:
```
Rails.application.config.spree.stock_splitters = []
```
5. If you want to add different splitters for each `StockLocation`, you need to decorate the
`Stock::Coordinator` class and override the `splitters` method

#### (3) The Prioritizer
* Decides which `StockLocation` should ship which package from an order to come up with the best
shipping situation available to the user
* By default, Prioritizer first selects packages where products are on hand. Then it finds packages
where items are backordered. During this, `Stock::Adjuster` is also used to ensure each package has
the correct number of items
* If you want to customize which packages take priority for the order you can override
`sort_packages` method in `Stock::Prioritizer`

##### Customizing the Adjuster
The `Adjuster` visits each package in an order and ensures the correct number of items are in each
package. To customize this functionality, you need to do two things:
    1. Subclass `Stock::Adjuster` class and override `adjust` method to get the desired functionality
    2. Decorate `Stock::Coordinator` and override `prioritize_packages` method, passing in your
    custom adjuster class to the `Prioritizer` initializer. For example:
```
Spree::Stock::Coordinator.class_eval do
  def prioritize_packages(packages)
    prioritizer = Prioritizer.new(order, packages, Spree::Stock::CustomAdjuster)
    prioritizer.prioritized_packages
  end
end
```

#### (4) The Estimator
`Stock::Estimator` loops through packages created by the packer to calculate and attach shipping
rates to them so the user so they can select shipments for their order
