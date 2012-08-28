## Associations

### Direct Relations
`:belongs_to` maps a model with a foreign key refers to another model's id.

`:has_one` maps a model whose id is the reference of another model's foreign key.

`:has_many` maps a model whose id is the reference of more than one models' foreign key.

`:has_and_belongs_to_many` uses a independent table to map many to many relations. You need to have a joining table with no primary key.

### Indirect Relations

`:has_many :through` maps a one-to-many relations and accesses the another model through a 3rd model.
It could also use to create shortcuts.

`:has_one :through` maps a one-to-one relation and accesses another model.

### Polymorphic Relations
Polymorphic likes setting up an interface to be used by multiple models.

By pass the options to make it polymorphic:

    class Comment < ActiveRecord::Base
      belongs_to :commentable, :polymorphic => true
    end

In the scheme, add two columns to store the references.

    class CreateComment < ActiveRecord::Migration
       def change
        create_table :pictures do |t|
          t.string  :name
          t.integer :commentable_id
          t.string  :commentable_type
          t.timestamps
        end
      end 
    end

    t.references :commentable, :polymorphic => true

In the 'owner' model, use `:as => commentable` to access the shared model.

    class Post < ActiveRecord::Base
      has_many :comments, :as => :commentable
    end

### Callbacks
Callbacks are available in the life cycle of a collection:
`before_add`
`after_add`
`before_remove`
`after_remove`

### Meta Methods
For `belongs_to :customer` or `has_one :customer`, these methods are provided to the object:

    customer
    customer=
    build_customer
    create_customer

`build_customer` is used to create association for existed object, `create_customer` to create a new one.

`has_many :orders` or `has_and_belongs_to_many` defines the following method:

    orders
    orders<<
    orders.delete
    orders=objects
    orders_singular_ids
    orders_singular_ids=
    orders.clear
    orders.empty?
    orders.size
    orders.find()
    orders.where()
    orders.exist?()
    orders.build
    orders.create


### Tricks
By default, association methods are built around caching, pass `true` to it to reload the cache. 

The join table infered by `has_and_belongs_to_many` use lexical order of class names to name the joining table.

Only module scope is applied for associations. To associate across modules, use `:class_name`

Use `:inverse_of` to provide ActiveRecord information between associations to avoid duplicate loading.

pass `:counter_cache` to cache the order in the reference model.

use `:foreign_key` to specify the column name directly.

use `:include` to eager load second-order association

use `:dependent` to trigger methods on the associated objects when this object is deleted.

When you assign object with `association=`, for a `has_one` relation, the object is instantly saved (in order to update its foreign key column); for a `belongs_to` relation, it doesn't save the object.
