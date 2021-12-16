## Active Record: How to select rows based on the number of items in an array column (PostgreSQL)

## Context

You have a column in your table that is used as an  [array](https://guides.rubyonrails.org/active_record_postgresql.html#array) , like this: 

```ruby
 t.string 'ids', array: true
```

And you want to select all records that have more than N elements in the array

## Solution

In PostgreSQL, you can use  [`cardinality`](https://www.postgresql.org/docs/current/functions-array.html)  to achieve this:

```
Model.where("cardinality(ids) > 5")
```







