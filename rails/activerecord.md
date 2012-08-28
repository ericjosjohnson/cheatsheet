## Concepts
ActiveRecord manipulates two **separate** things: `Ruby` objects and database objects.

To modify the database object, you will:
- create ActiveRecord object
- manipulate the attributes of the object
- save the attributes as a record

## Conventions and Configuration

### table names - class names
- the name of the table is the pluralized name of the class.
- table name is lowercase
- class name is CamelCase and table name is underscore.

### table creation
By default, each table will has an integer primary key named `id` which is automatically incremented.

### mapping
Columns of the table are mapped to the ActiveRecord instance's attributes.

To see the detailed column information use the `columns_hash[:name]` method.

It also maps the raw data type of those columns to attributes types. append `_before_type_cast` to attributes to see the raw type.

The column `type` is used to kept subclasses for ActiveRecord such as `Document::Release`.

`id` is always used to map the primary key even if the primary column doesn't name it.

The false concept in ActiveRecord is extended to `"", 0, "0", "false", "f"`.

## CRUD
### Create
`save` will be translated to `INSERT` to create and update data.

`create` accept an array of hash objects will call `save` implictly.

`first_or_create` will either create and fetch the first matched object, handy in use with `where` clause.

`first_or_initialize` is similar to `first_or_create` method.

### Read

`find` will be translated to `SELECT` to fetch data.

The first edition takes an argument for primary key:
`Account.find 5` translate to
`SELECT "accounts".* FROM "accounts" WHERE "accounts"."id" = $1 LIMIT 1  [["id", 1]]`

`Account.find 1, 2` translate to
`SELECT "accounts".* FROM "accounts" WHERE "accounts"."id" IN (1, 2)`

The second edition takes two argument, first for how many records to return, second is a hash options which will be translated to `WHERE` options.

`Account.find :all` translates to
`SELECT "accounts".* FROM "accounts"`

`Account.find :first` or `Account.first` translates to
`SELECT "accounts".* FROM "accounts" LIMIT 1`

`Account.last` translates to
`SELECT "accounts".* FROM "accounts" ORDER BY "accounts"."id" DESC LIMIT 1`

#### find options
`except()` could be used to except certain conditions.

`only()` could be used to filter conditions.

`.where()` or `:conditions` can be in form of an String, Hash or Array. `:conditions` can be in form of an String, Hash or Array.

`:conditions => ["username = ?", "peter"]` translates to
`WHERE (username = 'peter')`

`:conditions => {username: 'peter', id: 3}` translates to
`SELECT "accounts".* FROM "accounts" WHERE "accounts"."username" = 'peter' AND "accounts"."id" = 3`

It could even build up raw SQL in the Array form:
`:conditions => ["username = ? OR id = ?", 'peter', '3']` translates to
`SELECT "accounts".* FROM "accounts" WHERE (username = 'peter' OR id = '3')`

besides the `?` place holder, you could also use symbols.
`Client.where("created_at >= :start_date AND created_at <= :end_date", {:start_date => params[:start_date], :end_date => params[:end_date]})`

ranges could also be used in `where` conditions:
`Client.where(:created_at => (Time.now.midnight - 1.day)..Time.now.midnight)` translates to:
`SELECT * FROM clients WHERE (clients.created_at BETWEEN '2008-12-21 00:00:00' AND '2008-12-22 00:00:00') `


`order()` or `:order` will be add snippets to the `ORDER BY` SQL statements.
`order: 'id DESC, city'` translates to 
`ORDER BY id DESC`

`select()` and `:select` will be used to alter the columns to be fetched. `:select` will be used to alter the columns to be fetched.

You could append `.uniq` to fetch unique values in a certain field.

`:group` will be translated to `GROUP BY`.

`select: "length(city) as length, *"` will be translated to
`SELECT length(city) as length, * FROM ...`

`:from` allows to rewrite the entire content of the FROM clause.

`:limit` and `:offset` translate to the LIMIT and OFFSET in SQL.

`:readonly` to protect or relieve data write privilege, upon `save` will raise `ActiveRecord::ReadOnlyRecord`.

`:lock` to have database locks on selected rows.

#### Joins
`:join` append the string to the FROM clause of the SQL statement

ActiveRecord also allows you to use the names of associations for shortcut.
`Category.joins(:posts)` translates to:

`SELECT categories.* FROM categories
  INNER JOIN posts ON posts.category_id = categories.id`

`:include` is used to apply **Eager Loading** mechanism to reduce queries.
`:include` alter the `FROM` clause to automatically joining for you.

#### Scope
In the models, you could build queries as methods

    class Post < ActiveRecord::Base
      scope :published, where(:published => true)
    end

There could be `default_scope` specified in each model

The `scope` also accepts an lambda if run-time evaluation is needed.

You could store the queried objects with `.scoped` for later usage.

    client = Client.find_by_first_name("Ryan")
    orders = client.orders.scoped
    orders.where("created_at > ?", 30.days.ago)

Use `unscoped` to remove all scoping.

#### Dynamic find and method_missing
`find_by_<attr>[_and_<attr>]k`: find one record by attributes value.

`find_all_by_<attr>[_and_<attr>]`: find all records by attributes value.


`find_or_create_by_<attr>`: if a matching object is not found, one will be created.

#### Read Large Data
`find_each` will retrieve a batch of records batch by batch and yield them to the block. The default is 1000.

    Person.find_each(:batch_size => 2) do |people|
      puts people
    end  

will be break down to:

    SELECT "people".* FROM "people" WHERE ("people"."id" >= 0) ORDER BY "people"."id" ASC LIMIT 2
    SELECT "people".* FROM "people" WHERE ("people"."id" > 3) ORDER BY "people"."id" ASC LIMIT 2
    SELECT "people".* FROM "people" WHERE ("people"."id" > 5) ORDER BY "people"."id" ASC LIMIT 2

`find_in_batches` works similar to the `find_each` except it yields an array of models to the block.


#### Misc
`exist?` will query the database as always but return `true` or `false`.

`any?` is used to check existence on a model or relation.
`Post.recent.any?`
`Post.first.categories.any?`

method postfix with `!` will raise `RecordNotFound` error if record isnt found.

### Update
`save`, `update_attribute`, `update_attributes` will translate to the UPDATE in SQL.

`update_attribute` doesn't perform validation.

### Delete
`object.destroy` or `class.destroy(1, 23, 34)`, the second will have `destroy` call on object itself.

it will trigger callback or associated objects deletion as well.

`delete` works the same way except no callback is called.

The callback on associated object could be specified by the `:dependent` option of the Model.
if the value is `:destroy`, the child objects are loaded and destroyed with callbacks.
if the value is `:delete_all`, the child objects will just be deleted.

### Raw SQL
`find_by_sql` will execute SELECT SQL and return an array of object.

`connection#select_all` works similar to `find_by_sql` except no object is instantiated.

`execute` could be used to exec arbitrary SQL statement.

`pluck` accepts an single column name and returns the result in an raw Array.

## Calculations
calculation methods work on Model and relations.
`Person.count` results in 
`SELECT COUNT(*) FROM "people"`

Available method includes `average`, `mininum`, `maximum` and `sum`.

## Debug
Setup the logger to queries sent to the database:
    require 'logger'
    ActiveRecord::Base.logger = Logger.new(STDOUT)

use `explain` to inspect problematic queries.

It can be configed in `config.active_record.auto_explain_threshold_in_seconds`

## Transaction
`save` and `destroy` is wrapped in BEGIN and COMMIT by default.

rollback occurs when database or ActiveRecord complains.

The ActiveRecord objects are not updated when transaction fails.

Mannully create transactions through the `Class.transaction do ... end` block.

## Validations
`valid?` and `invalid?` to verify whether an object is valid or not.

**After validations** are executed, `errors` could be used to access errors. You could pass the attr as symbol to `errors`.

Error for a record is defined as **`errors` attributes are not empty**.

Methods trigger validations:

    create
    create!
    save
    save!
    update
    update_attributes
    update_attributes!

Common methods doesn't trigger validations:

    update_all
    update_attribute

Use `save!` and `create!` if we want to handle errors programmatically instead of popping up to the user.

### Validation Helpers
For `:message` options, the value could be accessed as `value`.

    Available helpers
    :acceptance => true
    :confirmation => true
    :exclusion => { :in => ['www', 'us', 'ca', 'jp'] }
    :inclusion
    :format => { :with => /\A[a-zA-z]+\z/}
    :length =>
      :minimum
      :maximum
      :in
      :is
    :numericality
    :presence
    :uniqueness
  
`:presence` use `blank?` to check and `blank?` takes `nil` and blank string as false. `false.blank?` is true.

### Validation Options
`:on` specifies when the validation happens, default to `:save`.
`:message` specifies the message added to the `errors`
`:allow_nil` and `:allow_blank` to bypass the validation.
`:if` and `:unless` accepts an symbol, a string or a Proc. They are used to determined conditions the validation to be called.
  symbol is the method name to be called.
  string is valid Ruby code to be evaled.
`with_options` is used to group conditions together for multiple validations.

### Customized Validator
`:validates_each` validates attributes against a block.

Custom validators are classes that extend `ActiveModel::Validator`.
They have a `validate` method and perform validation on an record, add `errors` if the record is invalid.
Custome validators are passed to a model through `:validates_with` method.

Custom individual validators attributes are inherited from `ActiveModel::EachValidator`.
They have a `validate_each` method.
They are passed to the model with corresponding name in options.

Custom methods could be created and passed to `validate` method.

### View Helpers
With `form_for` helpers, use `error_messages` and `error_messages_for` to access the error messages.

## Callbacks and Observers
### Callbacks

Important life events of an ActiveRecord object includes:

- (optional) find from db
- initialized
- validation
- saving to database
- create or update (to database)
- saved

Some common methods skip callbacks:
    delete
    delete_all
    find_by_sql
    update_all

Callbacks could be bound to relational statements through the `:dependent` options

Callbacks also accepts the `:if` and `:unless` conditions.

To reuse callbacks across multiple classes, encapsulate the callback methods in classes.

Two special callbacks related to tranactions are `after_commit` and `after_rollback`, these two callbacks doesn't pop up exceptions.

### Observers
When callback isn't directly related to the model.

Observers are simple ruby code in the `app/models` directory
They are registered through `config/application.rb`.

An observer could observe more than one models with `observe` method.

## Locking
Two versions of locking are supported by ActiveRecord.

### Optimistic Locking
the version you are saving is checked against the version in the database to make sure the model was not otherwise modified.

Use a column `lock_version` to track the object modification, raise `ActiveRecord::StaleObjectError` in version conflicts.

### Pessimistic Locking
row-level locking.

The first method is pass `:lock => true` to the `find()` method,
it will translate for FOR UPDATE.
A shortcut for this is `Class.lock.find`.

Usually to write this in a transaction to avoid deadlock.

To perform lock on an instance, use the `with_lock` block

    item.with_lock do
      # This block is called within a transaction,
      # item is already locked.
      item.increment!(:views)
    end


## Connection
Pass parameters to `ActiveRecord::Base.establish_connection` method:
    ActiveRecord::Base.establish_connection(
      :adapter => "postgresql", 
      :database => "test", 
      :host => "localhost",
      :username => "test",
      :password => "test") 
