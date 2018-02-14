# Column Defaults

A column default will be set by the database if we're creating a new record and when there's no value set for such a column.

## Adding Column Defaults

When adding a new column to a table, we can add a column default for that column:

```ruby
class AddDoneToTasks < ActiveRecord::Migration[5.1]
  def change
    add_column :tasks, :done, :boolean, default: false
  end
end
```

When migrating, the database will update all existing rows in the table and set the new column default:

```ruby
Task.first.done # => false
```

It will also be set when initializing new records and a value is omitted:

```ruby
Task.new.done # => false
Task.new(done: true).done # => true
```

Changing a column default using `change_column_default` will change the column default for new records but will not update any existing records.

## Defining Dynamic Column Defaults

Column defaults as shown above are static, when Rails adds them they will be quoted. If we want to use database functions as column defaults, we need to use a proc:

```ruby
class AddUuidToTasks < ActiveRecord::Migration[5.1]
  def change
    enable_extension 'uuid-ossp'
    add_column :tasks, :uuid, :uuid, default: -> { 'uuid_generate_v4()' }
  end
end
```

This will generate a new UUID in the database if there's no value set for the column.

Internally, Rails checks for the type of a column default. If it's a proc, it will use `proc.call` as the column default. If it's not a proc, it will quote the value and use that as the column default.

When using dynamic column defaults, Rails will not set the value when initializing new records:

```ruby
Task.new.uuid # => nil
```

Also, Rails will not return the generated value when creating new records:

```ruby
task = Task.create
task.uuid # => nil

task.reload
task.uuid # => '958f793e-91c9-4f0d-a8f0-f46d3d9478a3'
```

There's a closed issue for that, see [rails/rails#17605](https://github.com/rails/rails/issues/17605).

## Generated SQL Queries

If we're trying to save a record and the value for a column with a column default is either `nil` or exactly the column default, Rails will omit the value and will not send it at all:

```ruby
Task.create
# generates: INSERT INTO "tasks" ("created_at", "updated_at") VALUES ($1, $2) RETURNING "id"  [["created_at", "2018-02-14 18:35:21.637008"], ["updated_at", "2018-02-14 18:35:21.637008"]]

Task.create(done: false)
# generates: INSERT INTO "tasks" ("created_at", "updated_at") VALUES ($1, $2) RETURNING "id"  [["created_at", "2018-02-14 18:35:50.507602"], ["updated_at", "2018-02-14 18:35:50.507602"]]

Task.create(done: true)
# generates: INSERT INTO "tasks" ("created_at", "updated_at", "done") VALUES ($1, $2, $3) RETURNING "id"  [["created_at", "2018-02-14 18:36:06.606795"], ["updated_at", "2018-02-14 18:36:06.606795"], ["done", "t"]]
```

This is also the reason for the issue mentioned above: Rails is not telling the database to return dynamically generated column defaults when creating records; It's not necessary for static column defaults and not implemented for dynamic column defaults.
