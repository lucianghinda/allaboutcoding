## How to work with PostgreSQL enums in Rails 7

Rails 7 introduced a new way to create enums in PostgreSQL, by adding `create_enum` method. See [official documentation here](https://guides.rubyonrails.org/active_record_postgresql.html#enumerated-types).

I will explore the following tasks:
1. Creating an enum type and adding a column to a model with that enum type
2. How to add a new value for the enum
3. How to rename a value from enum type
4. How to delete a value from enum type

All examples were tested on `PostgreSQL 12`, `Ruby 3.1.2` and `Rails 7.1.0.alpha`

# How to create an enum type for an Active Record model

The migration will look like this:

```ruby
def up
  create_enum :user_status, ["pending", "active", "archived"]
  create_table :users, force: true do |t|
    t.enum :status, enum_type: "user_status", default: "pending", null: false
  end
end
```

The User model might look something like this: 

```ruby
class User < ActiveRecord::Base
  enum status: {
    pending: 'pending',
    active: 'active',
    archived: 'archived',
    disabled: 'disabled',
    waiting: 'waiting'
  }, _prefix: true
end
```

And then you can execute things like:

```ruby
User.status_pending.count # returning the number of Users with status pending
user = User.first
user.status_pending? # will return true if the status is `pending`
```

A rollback for this might look like this: 

```ruby
def down
  drop_table :users
  
  execute <<-SQL
    DROP TYPE user_status;
  SQL
end
```

# How to add a new value

You can do this by executing the following migration with raw SQL:

```ruby
disable_ddl_transaction

def up
  execute <<-SQL
      ALTER TYPE user_status ADD VALUE IF NOT EXISTS 'disabled' AFTER 'active';
    SQL
end
```

For creating a rollback, see the section about how to delete a value from an enum. Things are a bit more complicated.

# How to rename an enum type that a Rails model uses

Let's say you now want to rename `pending` to `waiting` and do that in Rails and the database. 

The migration might look something like the following:

```ruby
disable_ddl_transaction

def up
    execute <<-SQL
      ALTER TYPE user_status RENAME VALUE 'pending' TO 'waiting';
    SQL

    # don't forget to change the default status if you renamed the default one
    change_column_default :users, :status, from: 'pending', to: 'waiting'
end
```

# How to delete a value from an enum type

This is a more complex operation if you want to delete a value. 

First, you need to do the following two things:
1. Make sure you deleted or renamed any records that have the value that you want to delete in their enum column
2. Make sure you removed the default that points to the value you want to remove if you have that value as a default
3. Please notice that an enum type can be used for multiple tables, so you should check if other tables are using that enum and that specific value

Then your migration for this might look like this. Say I want to remove `waiting` status and replace it with `pending`

```ruby
def up
  # First, make sure no records are using the status that you want to remove
  User.status_waiting.update_all(status: 'pending')

  # change default to nil
  change_column_default :users, :status, nil

  execute <<-SQL
    --- Rename the old enum
    ALTER TYPE user_status RENAME TO user_status_old;

    --- Create the new enum as you will like it to be
    CREATE TYPE user_status AS ENUM('pending', 'active', 'archived');

    --- Alter the table to update it to use the new enum
    ALTER TABLE users ALTER COLUMN status TYPE user_status USING users::text::user_status;

    --- Drop the old status
    DROP TYPE user_status_old;
  SQL

  # make the default status pending
  change_column_default :users, :status, 'pending'
end

```

⚠️ This migration cannot be undone. You can write a rollback to recreate the enum that was initially, but that will not update back the Users that were in waiting. That information is lost.

If you want to keep that information, then the way to do this is:
1. You create a new status column named `backup_status` that should be of type `string` (because you probably don't want to create a new enum type, and you don't want to link the backup to the enum type that you are just about to change)
2. You create a migration that will copy `status` to `backup_status`
3. You can execute the migration that will remove the value `waiting` from the `user_status` enum as described above
4. You check if all is fine
5. Then you can delete the column `backup_status`

----
If you want to play with the code, I create a single file Rails app here: [`postgres_enums_in_rails_7.rb`](https://github.com/lucianghinda/shortrubynewsletter/blob/main/code-summaries/week37/postgres_enums_in_rails_7.rb) 

----
# Resources

1. https://guides.rubyonrails.org/active_record_postgresql.html#enumerated-types
2. https://blog.yo1.dog/updating-enum-values-in-postgresql-the-safe-and-easy-way/

---
If you like this content, follow me on Twitter [@lucianghinda](https://twitter.com/lucianghinda), where I tweet or retweet about Ruby and Ruby web frameworks.

I also publish a free newsletter with Ruby and Rails fresh content at [newsletter.shortruby.com](https://newsletter.shortruby.com) — in case you want to stay up to date with what is happening in Ruby and Rails world.



