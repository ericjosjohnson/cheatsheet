An inner database `schema_migration` table is maintained to record migration executed.

## Altering Methods
`ActiveRecord` provides methods to alter table, column and indexes:

All the methods are on ActiveRecord.connection object, they could be used to create temporary column, table or index.

- For tables `create_table`, `drop_table`, `change_table`
- For columns `add_column`, `change_column`, `remove_column`, `rename_column`
- For indexes `add_index`, `remove_index`

`execute` method takes a string and is used to execute arbitary SQL such as:

    execute <<-SQL
      ALTER TABLE products
        DROP FOREIGN KEY fk_products_categories
    SQL

[Available methods for scheme](http://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-distinct)
[Available for `create_table`](http://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/TableDefinition.html)
[Available for `change_table`](http://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/Table.html)

### Work with Columns

    :binary
    :boolean
    :date
    :datetime
    :decimal
    :float
    :integer
    :primary_key
    :string
    :text
    :time
    :timestamp

All type support this options:

    :null => true / false
    :limit => <size>
    :default => value

Number supports this options:

    :precision => 5
    :scale => 2

### Work with Tables
When create table with `create_table`, there are options available:

`:force => true` will drop an existing table and create a new one.

`:options => "xxxx"` to append SQL to the CREATE TABLE statement.

`:id => false` to turn off primary key, suitable for *join tables*.

### Work with indice

`add_index :table :column` method creates index for columns in table, it accepts
`:unique => true`
`:name => 'some_index_name'`

Pass an array to it to build composite index.

### Syntax Sugar & Shortcuts

    create_table :products do |t|
      t.column :name, string, :null => false
    end

would be written as:

    create_table :products do |t|
      t.string :name, :null => false
    end

`t.timestamps` will add `created_at` and `updated_at` columns.

`t.reference` and `t.belongs_to` will add a reference id columns.

## Command Line Tasks

`rails g migration <CamelCaseName>` will generate empty migration

`rails g migration Add/Remove***To*** column:type` will generate migration with `add_column` or `remove_column`

`rails g observer <ModelName>`

`rake db:migrate` runs the migrations which haven't been run.

`rake db:migrate:status` to check the current migration status.

`rake db:migrate VERSION=yyyymmddhhmmss` will run required migrations **until it reaches the target version**.

`rake db:rollback STEP=3` shortcuts for `rake db:migrate VERSION=***`

`rake db:migrate:redo STEP=3` shortcuts to first rollback and then apply the migrations again.

`rake db:reset` drop the databse and recreate it and load the `schema.rb`.

`rake db:structure:dump` to dump the SQL specific to the database.

## Caution
If needs to use Models in migrations, create a local model to avoid validation and verification conflicts.
    class AddFlagToProduct < ActiveRecord::Migration
      class Product < ActiveRecord::Base
      end

      def change
        Product.all each do |product|
          ...
        end
      end
    end
