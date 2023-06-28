---
title: "Ruby's range literals and their effect on Rails Active Record queries"
seoTitle: "Ruby's range literals and their effect on Rails Active Record queries"
seoDescription: "Investigate Ruby range literals in Rails Active Record queries, comparing SQL generation for inclusive/exclusive ranges"
datePublished: Wed Jun 28 2023 09:40:26 GMT+0000 (Coordinated Universal Time)
cuid: cljfj084m00050al7f85sgkl3
slug: rubys-range-literals-and-their-effect-on-rails-active-record-queries
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1687852997681/b8ceb6e1-0262-409c-a34f-c384ea7d6f88.png
tags: ruby, ruby-on-rails, active-record

---

The following Active Record query:

```ruby
User.where(created_at: 1.day.ago...Time.current).to_sql
```

will generate this SQL:

```sql
SELECT
  "users".*
FROM
  "users"
WHERE
  "users"."created_at" >= '2023-06-21 06:38:26.330063'
  AND 
  "users"."created_at" < '2023-06-22 06:38:26.330184'
```

While the following Active Record query:

```ruby
User.where(created_at: Date.current.all_day).to_sql
```

will generate the following SQL:

```sql
SELECT
  "users".*
FROM
  "users"
WHERE
  "users"."created_at" BETWEEN '2023-06-21 21:00:00' AND '2023-06-22 20:59:59.999999'
```

I noticed this difference while having a discussion with [Cosmin Stamate](https://twitter.com/monovertex) and [Jakob Cosoroabă](https://twitter.com/jcsrb) on a Discord group about data ranges.

Thus I started asking myself:

* Why is there a difference?
    
* Why use `BETWEEN` in the second example or why not use it in the first example?
    

And had a hint that it must be related to ranges and the inclusion/exclusion of their ends.

To answer these questions, we need to explore the following concepts:

1. What is `BETWEEN` doing in PostgreSQL?
    
2. What does `.all_day` do?
    
3. What is the difference between `...` and `..` ?
    

### PostgreSQL `BETWEEN`

This is straightforward. According to the [PostgreSQL documentation](https://www.postgresql.org/docs/current/functions-comparison.html) on function comparisons, BETWEEN includes its endpoints:

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687489677305/3d317949-9371-444d-9ef6-3e4f5a207b40.png align="center")](https://www.postgresql.org/docs/current/functions-comparison.html)

### What does `all_day` do?

`.all_day` seems to be [defined in ActiveSupport](https://github.com/rails/rails/blob/main/activesupport/lib/active_support/core_ext/date_and_time/calculations.rb#L310):

```ruby
# Source: https://github.com/rails/rails/blob/main/activesupport/lib/active_support/core_ext/date_and_time/calculations.rb#L310
def all_day
  beginning_of_day..end_of_day
end
```

So it is a [range literal](https://docs.ruby-lang.org/en/3.2/syntax/literals_rdoc.html#label-Range+Literals) that includes its end value.

### What is the difference between `...` and `..` ?

Here is what Ruby 3.2 documentation defines [Range Literals](https://docs.ruby-lang.org/en/3.2/syntax/literals_rdoc.html#label-Range+Literals):

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687490091376/f1580101-8ebb-4a15-ace5-4e864f02ec18.png align="center")](https://docs.ruby-lang.org/en/3.2/syntax/literals_rdoc.html#label-Range+Literals)

A range has a starting point and an endpoint. Different from the math notation of using `(.)` to signify that the endpoints are not included and `[.]` to signify that the endpoints are included, in Ruby both `..` and `...` include the start point.

The difference between them is with regards to the endpoint:

* `..` includes its ending value
    
* `...` does NOT include its ending value
    

```ruby
(1..2).to_a # => [1, 2]
(1...2).to_a # => [1]
```

They even have a method to tell you that:

```ruby
(1..2).exclude_end? # => false
(1...2).exclude_end? # => true
```

### So, why is the difference in SQL for `.all_day` vs `1.day.ago ...Time.current`

The difference is that `all_day` uses an inclusive end range and `1.day.ago ... Time.current` is an exclusive end range.

```ruby
(1.day.ago...Time.current).exclusive_end? # => true
Date.current.all_day.exclusive_end? # => false
```

Thus in the case when using an inclusive end range, Active Record will generate an SQL statement with `BETWEEN` and when using an exclusive end range it will use the normal comparison `>= AND <` SQL statement.

Let's make the calls explicit. Here is the SQL query when using the exclusive end range:

```ruby
User.
  where(created_at: 1.day.ago.beginning_of_day ... 1.day.ago.end_of_day).
  to_sql

# will generate the following statement
# please keep in mind this is executed with 
# Europe/Bucharest set as time zone

SELECT "users".* FROM "users" 
    WHERE 
    "users"."created_at" >= '2023-06-21 21:00:00' 
    AND 
    "users"."created_at" < '2023-06-22 20:59:59.999999'
```

Here is the query using inclusive end range:

```ruby
User.
  where(created_at: 1.day.ago.beginning_of_day .. 1.day.ago.end_of_day).
  to_sql

# will generate the following statement
# please keep in mind this is executed with 
# Europe/Bucharest set as time zone

SELECT "users".* FROM "users" 
    WHERE 
    "users"."created_at" BETWEEN '2023-06-21 21:00:00' AND '2023-06-22 20:59:59.999999'
```

### What about using AREL?

It will do the same thing:

```ruby
users = User.arel_table
users.
  project(Arel.star).
  where(
    users[:created_at].between(1.day.ago.beginning_of_day ... 1.day.ago.end_of_day)
  ).to_sql

# will print

SELECT "users".* FROM "users" 
    WHERE 
    "users"."created_at" >= '2023-06-21 21:00:00' 
    AND 
    "users"."created_at" < '2023-06-22 20:59:59.999999'

# while using inclusive end range

users.
  project(Arel.star).
  where(
    users[:created_at].between(1.day.ago.beginning_of_day .. 1.day.ago.end_of_day)
  ).to_sql

# will print

SELECT "users".* FROM "users" 
    WHERE 
    "users"."created_at" BETWEEN '2023-06-21 21:00:00' AND '2023-06-22 20:59:59.999999'
```

---

Enjoyed this article?

Join my [**Short Ruby News**](https://shortruby.com/) **newsletter** for weekly Ruby updates. Also, check out my co-authored **book**, [**LintingRuby**](https://lintingruby.com/), for insights on automated code checks. For more Ruby **learning resources**, visit [**rubyandrails.info**](http://rubyandrails.info)**.**