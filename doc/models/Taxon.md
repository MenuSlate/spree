## Taxon (Model)
* Single child node that exists at a given point within a `Taxonomy`
* Each can contain many or no child taxosn
* Ordered by its `position` attribute
* Uses [Nested set model](http://en.wikipedia.org/wiki/Nested_set_model) for hierarchy. `lft` and `rgt` columns in `spree_taxons` table represent locations within the hierarchy of the item (all handled by `awesome_nested_set` gem)
* Links to products through a `Classification` model that:
  * When a product is deleted all links from it to its taxons are deleted automatically
  * When a taxon is deleted all links to products are deleted automatically
* Admins can link a product to multiple Taxons within a `Taxonomy`
* Linking to a taxon in a controller or a template should be done using `nested_taxons_path` helper which will use the taxon's permalink to generate a URL such as `/t/categories/brand`

> See [Taxonomy](../models/Taxonomy.md)

#### Attributes
* `parent_id`
* `position`
* `name`: A required field for all taxons. The name determines what the user will see when they 
look at this product in your store
* `permalink`: The end of the URL a user goes to, in order to see all products associated with this 
taxon. This field is also required, and a value is automatically generated for you when you 
create the taxon. Be careful with arbitrarily changing the permalink - if you have two taxons 
with the same permalink you will run into issues
* `taxonomy_id`
* `lft`
* `rgt`
* `icon_file_name`: Icons are currently not functional
* `icon_content_type`
* `icon_file_size`
* `icon_updated_at`
* `description`: currently not functional
* `meta_title`: Overrides the store's setting for page title when a user visits the taxon's  page
 on the front end of the website
* `meta_description`: Overrides the store's setting for meta description when a user visits the 
taxon's page on the front end of the website
* `meta_keywords`: Overrides the store's setting for meta keywords when a user visits the taxon's
 page on the front end of the website
* `depth`: depth of the taxon within the hierarchy, e.g. the most *parent* depth is 0




### Attributes `products_taxons`
* `product_id`
* `taxon_id`
* `position`
