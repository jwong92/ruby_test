# Active Record Migrations
Migrations used to alter database schema over time. Use Ruby DSL to allow schema and changes to be independent. Each migration is a new 'version' of the database. Schema begins with nothing in it, and each migration modifies it to add or remove tables, columns or entries.

```Example of migration
class CreateProducts < ActiveRecord::Migration[5.0]
  def change
    create_table :products do |t|
      t.string :name
      t.text :description
 
      t.timestamps
    end
  end
end
```
* This migration adds table products with a string column called name and text column called description.
* Primary keys are added implicitly
* Timestamp macro's adds two columns: created_at and updated_at.

## Creating a Migration
### Creating Standalone Migrations
Migrations stored as files in db/migrate directory for each migration class.
```
class AddPartNumberToProducts < ActiveRecord::Migration[5.0]
  def change
  end
end
```
* This creates an empty migration which you can technically modify
* `$ bin/rails generate migration AddPartNumberToProducts part_number:string`
  * This will generate
  ```
  class AddPartNumberToProducts < ActiveRecord::Migration[5.0]
      def change
          add_column :products, :part_number, :string
      end
  end
  ```
* `$ bin/rails generate migration CreateProducts name:string part_number:string`
  * This will generate
  ```
  class CreateProducts < ActiveRecord::Migration[5.0]
    def change
      create_table :products do |t|
        t.string :name
        t.string :part_number
      end
    end
  end
  ```
* `$ bin/rails generate migration AddUserRefToProducts user:references`
  * This accepts column types as references (aka belongs_to)
  ```
  class AddUserRefToProducts < ActiveRecord::Migration[5.0]
    def change
      add_reference :products, :user, foreign_key: true
    end
  end
  ```
  * This will create a user_id column and appropriate index.
  [1]("Api Documentation", http://api.rubyonrails.org/v5.2.0/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-add_reference)

* `$ bin/rails g migration CreateJoinTableCustomerProduct customer product`
  * This will join tables:
  ```
  class CreateJoinTableCustomerProduct < ActiveRecord::Migration[5.0]
    def change
      create_join_table :customers, :products do |t|
        # t.index [:customer_id, :product_id]
        # t.index [:product_id, :customer_id]
      end
    end
  end
  ```

  ### Model Generators
  These create migrations appropriate for adding a new model. This migration already contains instructions for creating the relevant table.
  * `$ bin/rails generate model Product name:string description:text`
    * This creates a migration like:
    ```
    class CreateProducts < ActiveRecord::Migration[5.0]
      def change
        create_table :products do |t|
          t.string :name
          t.text :description
    
          t.timestamps
        end
      end
    end
    ```
## Writing a Migration
Note that this is essentially modifying the migration file itself.

### Creating a Table
__create_table__ method typical use is:
```
create_table :products do |t|
  t.string :name
end
```
* Creates a `products` table with column name and implicit id column
* You can change the name of the primary key with `:primary_key` option, or pass `id: false` for no primary key
* Can also pass the `:comment` option with any description for the table to be stored in the database itself.
  ```
  create_table :products, options: "ENGINE=BLACKHOLE" do |t|
    t.string :name, null: false
  end
  ```

### Creating a Join Table
__create_join_table__ creates a has and belongs to many join table
  * `create_join_table :products, :categories` is a use case.
    * Creates a categories_products table with two columns: category_id and product_id.
    * By default, name of the join table is from the union of first two arguments provided by create_join_table in alphabetical order. Cutomize name with `:table_name` option.
    ```
    create_join_table :products, :categories do |t|
      t.index :product_id
      t.index :category_id
    end
    ```

### Changing Tables
Used for changing existing tables. Used in a similar fashion to create_table but the object yielded in the block knows more methods.
```
change_table :products do |t|
  t.remove :description, :name
  t.string :part_number
  t.index :part_number
  t.rename :upccode, :upc_code
end
```
* This removes the description and name columns, creates a part_number string column and adds an index. It also renames the upccode column

### Changing Columns
* `change_column :products, :part_number, :text`
* This changes the column part_number on the products table to be a :text field. Note that this is __irreverssible__
* To change other constraints and default values, you can use __change_column_null__ and __change_column_default__
```
change_column_null :products, :name, false
change_column_default :products, :approved, from: true, to: false
```
  * This sets :name field on products to a NOT NULL column and default value of the :approved field from true to false.

### Column Modifiers
Applied when creating or changing a column
* __limit__ - sets the max size of a string/text/binary/integer fields
* __precision__ - defines the precision for decimal fields representing total number of digits in the number
* __scale__ - defines scale for the decimal fields to represent number of digits after the decimal point
* __polymorphic__ - adds a type column for __belongs_to__ association
* __null__ - allows or disallows null values in the column
* __default__ - allows setting of a default value on the column
* __index__ - adds an index for the column
* __comment__ - adds a comment for the column
__null__ and __default__ can't be specified via command line.

### Foreign Keys
Not required, but guarantees referential integrity
* `add_foreign_key :articles, :authors`
  * Adds a new foreign key to the author_id column of the articles table. The key references the id column of the authors table. If the column names can't be derived from the table names, can use the __:column__ and __:primary_key__ options.
  * Names for the fk are auto generated, but can be specified with the __:name__ option if needed.
  ```Some more examples
  add_foreign_key :accounts, column: owner_id
  remove_foreign_key :accounts, name: :special_fk_name
  ```

### Execute Method
If helpers aren't enough, can use the __execute__ method to execute arbitrary SQL.
  * `Product.connection.execute("UPDATE products SET price = 'free' WHERE 1=1")`

### Reverting Previous Migrations
Use active records ability to rollback migrations using the __revert__ method.
```
require_relative '20121212123456_example_migration'
 
class FixupExampleMigration < ActiveRecord::Migration[5.0]
  def change
    revert ExampleMigration
 
    create_table(:apples) do |t|
      t.string :variety
    end
  end
end
```
* Revert accepts a block of instructions to reverse.

## Running Migrations
There are bin/rails tasks to run certain sets of migrations.
  * `rails db:migrate` runs the change or up method for all the migrations that have not yet been run. If there are no such migrations, it exits.
    * Running this task invokes the `db:schema:dump` task which ill update the db/schema.rb file to match the structure of the database.
  * Can specify a target version and Active Record will run the required migration until it reaches the specified version. The version is the numerical prefix on the migrations filename.
    * `$ bin/rails db:migrate VERSION=20080906120000`

### Rolling Back
Rolling back the last migration rather than tracking down the associated version number.
  * `$ bin/rails db:rollback`
    * Rollbacks the latest migration by reverting the change method or running the down method.

To undo several migrations, provide a STEP parameter
  * `$ bin/rails db:rollback STEP=3`
    * Where the number associated with the step determines the number of migrations to rollback.
  * `$ bin/rails db:migrate:redo STEP=3`
    * This is a shortcut for doing a rollback and then re-migrating up again.

### Setting up the Database
* `rails db:setup` task creates the database, loads the schema and initilizes it with the seed data. 
* `rails db:reset` task drops the database and sets it up again.

### Running migrations on different environments
* By default, migrations run in the development environment. To run migrations in another environment you can specify it using the RAILS_ENV variable which running the command.
  * `$ bin/rails db:migrate RAILS_ENV=test`

## Changing Existing Migrations
First, rollback the migration
  * `bin/rails db:rollback`
Edit the migration, then run
  * `rails db:migrate`

## Schema Dumps
Change the schema format in __config/application.rb__ by the __config.active_record.schema_format__ setting.
  * This may be set to either :sql or :ruby
  * `pg_dump` utility or `db:structure:dump`

## Active Record and Referential Integrity
Active records maintains that intelligence belongs in models, not the database. So, features such as triggers or constraints that push intelligence back into the database aren't heavily used.

Validations such as `validates :foreign_key` or `uniqueness: true` are ways which models can enforce data integrty.

## Using Seed Data
```
class AddInitialProducts < ActiveRecord::Migration[5.0]
  def up
    5.times do |i|
      Product.create(name: "Product ##{i}", description: "A product.")
    end
  end
 
  def down
    Product.delete_all
  end
end
```
* After creating this migration, run `rails db:seed`
* This is a clean way to set up the database of a blank application.