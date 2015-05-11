## Product
* `Product` track sellable entities within a store
* Products can have unique variations, called variants
* Products Properties are distinguishing properties that don't change by variant such as
description, permalink, availability, shipping category, etc.

### Attributes:
* `name`
* `description`
* `permalink`: An SEO slug based on the product's name and is placed into its URL
* `available_on`: first date product is available for sale. If not set it will not appear for sale
* `deleted_at`: date the product is no longer available for sale
* `meta_description`: description targeted at search engines
* `meta_keywords`: words/phrases targeted at search engines

## Option Types and Option Values
* Option types are different options that distinguish variants
* A product must be assigned at least one option type to create variants out of it

#### Examples
* Option Type: size
* Option Values: "Small", "Medium", "Large"

### Master Variants
* Created whenever a new product is created so every product has a single master variant
* Stores master price and sku, size and weight, etc.
* Stores on_hand inventory levels/units only when there are no variants for the product
* Does not have option values associated with it
* Master variants sole purpose is having a consistent API when associating variants and
`orders#line-items`. Otherwise line items would need to track a polymorphic association which can
 be a product or a variant

## Variants
* A variant tracks a group of individual properties (Option Types) of a `Product` such as height,
 width, depth, and cost price
* These properties are unique to each variant and so are on top of `Product Properties` that apply
 to all variants of a product
* All variants can access the product properties directly (via reverse delegation)
* Inventory units are tied to Variant. Other variants have option values and may have inventory units
* Sum of on_hand each variant's inventory level determine "on_hand" level for the product
* `sku` field holds product's stock code

### Normal Variants
All non-master variants are  based on `option_type` and `option_value` combinations
#### Example
* Product: Baseball Jersey
* Option Type: size
* Option Values: "Small", "Medium", "Large"
* Option Type: color
* Option Values: "Red", "Green", "Blue"
* Variants: "Small, Red", "Small, Green", "Small, Blue", "Medium, Red", "Medium, Green"...

## Images
* Images link to a product through its master variant.
* sub-variants may also have their own unique images
* Creation and storage of several size versions of each image is handled via the Paperclip plugin:
```
:styles => {
  :mini => '48x48>',
  :small => '100x100>',
  :product => '240x240>',
  :large => '600x600>'
}
```
* Image sizes can be changed by altering the value of `Config[:attachment_styles]`and regenerating
the paperclip thumbnails with this Bash command:
```
$ bundle exec rake paperclip:refresh:thumbnails CLASS=Spree::Image
```
* To change the image displayed when a product has no image, create new versions of the files within
[app/assets/images/noimage]

## Product Properties
* Tracks individual attributes for a product that don't apply to other products, typically
information that wouldn't create new variants
* You can retrieve a property value on a `Product` by calling `property` and passing its name:
```bash
$ product.property("material")
=> "100% Cotton"
```
* You can set a property on a product by calling the `set_property` method:
```
product.set_property("material", "100% cotton")
```

#### Examples
* Product: T-Shirt
* Property: "material", "fit type"

## Multi-Currency
* `Price` objects track a price for a particular currency and variant combination. For instance,
a Variant may be available for $15 and €7
* Presence or lack of a price for a variant in a currency will determine if the variant is visible
in the frontend. If no variants of a product have a particular price value for the site's current
currency, that product will not be visible in the frontend.
* You may see what price a product would be in the current currency (`Config[:currency]`) by calling
the `price` method on that instance:
```bash
$ product.price
=> "15.99"
```
* To find the currencies a product is available in, call `prices` to get related `Price` objects:
```bash
$ product.prices
=> [#<Spree::Price id: 2 ...]
```

## Prototypes
A way to share `OptionType` and `Property` combinations amongst different products. For instance,
if you're creating a lot of shirt products, you may wish to maintain the "Size" and "Color" option
types as well as a "Fitting Type" property

## Taxons and Taxonomies
* `Taxonomy` – hierarchical list made up of individual Taxons. Each taxonomy relates to one `Taxon`
that is its root node
* `Taxon` – a single child node which exists at a given point within a `Taxonomy`. Each `Taxon` can
contain many (or no) sub / child taxons.
* Admins can define many Taxonomies and link a product to multiple Taxons from each Taxonomy
* By default, both Taxons and Taxonomies are ordered by their `position` attribute
* Taxons use [Nested set model](http://en.wikipedia.org/wiki/Nested_set_model) for their hierarchy.
The `lft` and `rgt` columns in the `spree_taxons` table represent the locations within the hierarchy
of the item. This logic is handled by awesome_nested_set gem
* Taxons link to products through a `Classification` model that when a product is deleted, all links
from it to its taxons are deleted automatically. Similarly, when a taxon is deleted all links to
products are deleted automatically
* Linking to a taxon in a controller or a template should be done using `nested_taxons_path` helper
which will use the taxon's permalink to generate a URL such as `/t/categories/brand`
