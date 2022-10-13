# Alternative ways to freeze a string in Ruby

If you want to freeze strings in Ruby there are at least two ways to do this:



1) Adding the magic comment at the beginning of the file

```ruby
# frozen_string_literal: true
```

2) Calling `.freeze` on the string that you want to freeze

```ruby
a = "this is a frozen string".freeze
puts a.frozen? # will return true 

b = "this is a frozen string".freeze
puts b.frozen? # will return true 

puts a.equal?(b) # will return true
puts a.object_id == b.object_id # true
```

As you notice `a` and `b` seem to be the same object instance. 
No new objects are instantiated. 

The same happens for symbols: 

```ruby
s = :a_new_symbol_open
puts s.frozen? # will, of course return true

m = :a_new_symbol_open
puts m.frozen? # will, of course return true

puts s.object_id == m.object_id # will return true
```

## Alternative ways to freeze a String

Enter a kind of strange method that can be applied on Strings: `-`

You might see code that looks like this:

```ruby
status = -"global.pending"
```


## `String#-@` and `String#+@` methods

Let's explore how these methods work. 

Using a string literal will create a new String object every time an assignment takes place. Notice in the following example that the object_id is different between `str1` and `str2`:

```ruby
str1 = "Normal string"
puts "#{str1.object_id}, #{str1.frozen?}" # 60, false

str2 = "Normal string"
puts "#{str2.object_id}, #{str2.frozen?}" # 80, false

puts str1.object_id == str2.object_id # false
```

### What does using `-` on a String do?

First, [here](https://docs.ruby-lang.org/en/3.1/String.html#method-i-2D-40) is the definition of `String#-`

> Returns a frozen, possibly pre-existing copy of the string

> The returned String will be deduplicated if it has no instance variables.

Notice in the following example that the strings are frozen and return the same object_id:

```ruby
str3 = -"Normal string"
puts "#{str3.object_id}, #{str3.frozen?}" # 100, true

# Here for example, it will return the same object id
str4 = -"Normal string"
puts "#{str4.object_id}, #{str4.frozen?}" # 100, true

puts str3.object_id == str4.object_id # true
```

Thus we are not only making the string close to modifications but we are also re-using the same object. 


### How to unfreeze such string?

There is a counter-part method on String: `+`

It [does](https://docs.ruby-lang.org/en/3.1/String.html#method-i-2B-40) the following:

> Returns a frozen, possibly pre-existing copy of the string.

> The returned String will be deduplicated as long as it does not have any instance variables set on it.

```ruby
frozen_string = -"This is a frozen string"
begin
  frozen_string << "and it cannot be modified"
rescue FrozenError => e
  puts e # can't modify frozen String: "This is a frozen string"
end 

puts "#{frozen_string.object_id}, #{frozen_string.frozen?}" # 120, true

str5 = +frozen_string 
puts "#{str5.object_id}, #{str5.frozen?}"  # 140, false

str5 << " and it can be modified" # This is a frozen string and it can be modified
puts str5
```

## Exploring some interesting cases

Hashes with String keys in Ruby are freezing the keys. But are the keys the same object? What about values?

```ruby
hash1 = { "Key" => "Value" }
key1 = hash1.keys[0]
value1 = hash1.values[0]
puts "H1: #{key1.object_id}, #{value1.object_id}" # 160, 180
puts "H1: #{key1.frozen?}, #{value1.frozen?}" # true, false


hash2 = { -"Key" => "Value" }
key2 = hash2.keys[0]
value2 = hash2.values[0]
puts "H2: #{key2.object_id}, #{value2.object_id}" # 160, 200
puts "H2: #{key2.frozen?}, #{value2.frozen?}" # true, false

puts key1.equal?(key2) # true
puts key1.object_id == key2.object_id # true
```

As you can notice, the key has the same object_id. Thus, it is the same object instance. But values have different object_id, so they are not the same object.

```ruby
puts value1.equal?(value2) # false
puts value1.object_id == value2.object_id # false
```

So if you want to make also the value the same object as it is the same string literal: 

```ruby
hash3 = { "Key" => -"Value" }
key3 = hash2.keys[0]
value3 = hash2.values[0]
puts "H3: #{key3.object_id}, #{value3.object_id}" # 160, 200
puts "H3: #{key3.frozen?}, #{value3.frozen?}" # true, true

hash4 = { "Key" => -"Value" }
key4 = hash2.keys[0]
value4 = hash2.values[0]
puts "H4: #{key4.object_id}, #{value4.object_id}" # 160, 200
puts "H4: #{key4.frozen?}, #{value4.frozen?}" # true, true

puts value3.equal?(value4) # true
puts value3.object_id == value4.object_id # true
```

## Some examples of using the `-` on strings

First when possible, use symbols instead of strings. 

Of course, symbols have a bit more restrictive rules to write, so it might be that for clarity or other reasons you cannot/don't want to use symbols. 

So use `-` on strings that will probably not have variations, not they will be modified

Examples:

1) When writing a custom SQL that might be re-used in some other places

```ruby
SolarSystem.planets.order(-"orbit = 'circular' ASC, name ASC")
```

2) When calling an external API where you define the base URL or the path:

```ruby
base = -"http://example.com"
url = URI.join(base, -"/foo")
```

3) When adding, for example headers:

```ruby
header = { "Content-Type" => -"application/x-www-form-urlencoded" }
```

4) When you want to log some structured short information:

```ruby
Rails.logger(-"job.start.highpriority")
```
