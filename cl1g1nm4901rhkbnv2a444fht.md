## How to update multiple different values at the same time in one query


# Context

You want to update the same column for multiple records in one query and set different values for each record.

Example:
```ruby
class Car < ActiveRecord::Base
# has attributes: id, license_plate
end
```

Let's say you want to update license plates and you have a list of car ids and associated license plates:

```ruby
updates = {
  1 => "EU-DE-123245"
  2 => "EU-NL-12456"
  3 => "EU-FR-12455"
  4 => "EU-RO-87613"
}```

# Problem

You can do this in multiple queries. Here is an example: 

```ruby
updates.each do |car_id, license_place|
  Car.find(car_id).update_column(:license_place, license_place)
end
```

The problem with this solution is that it will do multiple queries and sometimes this could come with a performance penalty.

So you want to find a way to update all records with different values in one single query.

# Solution

```ruby
# @param column [String] The name of the column to be updated with new values
# @param updates [Array<Hash>] The hash key is the id of the record and hash value is the new value to be set
# @example Update amount for records with id 1 and 2 
#     update_column_for_all_records("amount", [ {
#       1 => "EU-DE-123245"
#       2 => "EU-NL-12456"
#       3 => "EU-FR-12455"
#       4 => "EU-RO-87613"
#     }])

def update_column_for_all_records(column, updates)
  cases = []
  ids = updates.keys.map(&:to_i)

  Car.where(id: ids).each do 
    id = _1.id
    new_value = updates[id]
    cases << sanitize_case(id, new_value) 
  end
  
  update_query = "#{column} = CASE id #{cases.join(" ")} END"

  Car.where(id: ids).update_all(update_query)
end

def sanitize_case(id, new_value)
  Car.sanitize_sql [ "WHEN :id THEN :value", { id: id, value: new_value} ]
end
```

## Explanation of the solution

We are using a kind of raw SQL to update all records in one query.

What we want to execute in SQL is something like this:

```sql
UPDATE
	"cars"
SET
	"cars"."license_plate" = CASE "cars"."id"
	WHEN 1 THEN
		'EU-DE-123245'
	WHEN 2 THEN
		'EU-NL-12456'
	END
WHERE
	"cars"."id" IN(1, 2)
```
# Things to consider

If you use this in #Rails please take into consideration: 

## `update_all` will not execute callbacks nor validations
As specified in the [Rails docs](https://api.rubyonrails.org/classes/ActiveRecord/Relation.html#method-i-update_all), update_all: 

>  It does not instantiate the involved models and it does not trigger Active Record callbacks or validations. 

Thus it will also **not update** the `updated_at` column. 
  
## You might need to cast your values in the WHEN THEN query

For example, if you will need to update a column that has the format `timestamp` then you might need to change the method to support casting and add to the `cases` array something like: 

```ruby
cases << "WHEN #{id} THEN CAST('#{new_value.to_fs(:db)}' as timestamp)"
``` 

Notice in this line the expectation is that `new_value` is a Rails DateTime object and I am using `to_fs(:db)` (see more [here](https://api.rubyonrails.org/classes/DateTime.html#method-i-to_fs)) to make sure I am converting the proper value to timestamp. 

Here is a more full example with casting:

```ruby
# @param column [String] The name of the column to be updated with new values
# @param updates [Array<Hash>] The hash key is the id of the record and hash value is the new value to be set
# @example Update amount for records with id 1 and 2 
#     update_column_for_all_records("amount", [{ id: 1, value: "EU-DE-123245", type: "string" }, {id: 2, value: "EU-NL-12456", type: "text"}])
    
def update_column_for_all_records(column, updates)
  cases = []
  ids = updates.map { _1[:id] }

  Car.where(id: ids).each do 
    id = _1[:id]
    new_value = _1[:value] 
    cases << sanitize_case(id, new_value, _1[:type]) 
  end
  
  update_query = "#{column} = CASE id #{cases.join(" ")} END"

  Car.where(id: ids).update_all(update_query)
end

def sanitize_case(id, new_value, type)
  native_type = adapter_type(type)
  Car.sanitize_sql [ "WHEN :id THEN CAST(:value as #{native_type})", { id: id, value: new_value} ]
end

# This works for PostgreSQL for example
def adapter_type(type)
  ActiveRecord::ConnectionAdapters::PostgreSQLAdapter::NATIVE_DATABASE_TYPES.dig(type.to_s, :name) || "text"
end
```

