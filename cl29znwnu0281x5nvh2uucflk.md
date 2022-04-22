## Ruby: What does && and || return?

# How do these things work?

Have you ever wondered how the following code works? Specifically, how come the key will have another value than true or false. 

```ruby
key = loaded_key || Key.find(params[:key])
```

or maybe how does the simple memorization pattern work: 

```ruby
def key
  @key ||= Key.find(params[:key])
end
```

The answer is in the idea that `||` and `&&` do not return the response as`TrueClass` or `FalseClass` objects (or, more precisely, do not return the global value `true` or `false`) but it returns one operand or the other. 

# What is false and what is true in Ruby

`nil` and `false` are the only `falsy` values. 

Everything else is considered `truthy` ([source](https://www.ruby-lang.org/en/documentation/faq/6/))

# What does && return

```ruby
x && y
# returns x if x is false
# returns y if x is true
```
A simple truth table might look like this: 

```ruby
# +────────+─────────────+─────────+
# | X      | Y           | Return  |
# +────────+─────────────+─────────+
# | false  | true/false  | X       |
# | true   | true        | Y       |
# | true   | false       | Y       |
# +────────+─────────────+─────────+

```

But as in Ruby, we have the concept of `falsy` values, let's expand this table:

```ruby
# +──────────+─────────+─────────+─────────+
# | X        | Y       | Result  | Return  |
# +──────────+─────────+─────────+─────────+
# | nil      | Truthy  | nil     | X       |
# | nil      | Falsy   | nil     | X       |
# | false    | Truthy  | false   | X       |
# | false    | nil     | false   | X       |
# | true     | Falsy   | Falsy   | Y       |
# | Truthy   | Truthy  | Truthy  | Y       |
# +──────────+─────────+─────────+─────────+

```
When evaluating `object_a && object_b` Ruby will return:
- `object_a` if `object_a` is a falsy value (either `nil` or `false`)
- `object_b` if both object_a and object_b are truthy values (not `nil` and not `false`)
- `object_b` if `object_a` is truthy and `object_b` is falsy

You can check that by running the examples I added in this Github gist:
https://gist.github.com/lucianghinda/2677e6549c9e5927bd88d9c646524564

# What does || return

```ruby
x || y 
# returns x if x is true without evaluating y
# returns y if x is false
```
Let's try to put this in a kind of truth table: 

```ruby
# +───────+─────────────+─────────+
# | X     | Y           | X || Y  |
# +───────+─────────────+─────────+
# | true  | true/false  | X       |
# | false | true/false  | Y       |
# +───────+─────────────+─────────+
```

And then to expand it by taking into consideration the concept of `truthy` and `falsy` from Ruby:

```ruby
# +────────+─────────+─────────+─────────+
# | X      | Y       | Result  | Return  |
# +────────+─────────+─────────+─────────+
# | nil    | Truthy  | Truthy  | Y       |
# | nil    | Falsy   | Falsy   | Y       |
# | false  | Truthy  | Truthy  | Y       |
# | false  | Falsy   | Falsy   | Y       |
# | Truthy | Falsy   | Truthy  | X       |
# | Truthy | Truthy  | Truthy  | X       |
# +────────+─────────+─────────+─────────+
```

We can then say that `object_a || object_b` will return:

- `object_a` if `object_a` is truthy (not `nil` and not `false`)
- `object_b` if `object_a` is falsy (either `nil` or `false`)


# Explaining how the `||=` work

Coming back to the question asked at the beginning, how does the following piece of code work:

```ruby
key = loaded_key || Key.find(params[:key])
```

We can read it like this:

- will return `loaded_key` if `loaded_key` is not `nil` and not `false`
- will return `Key.find(params[:key])` if `loaded_key` is `nil` or `false`.

What about **memoization**:

```ruby
def key
  @key ||= Key.find(params[:key])
end
```

First this code is actually a shorthand for: 

```ruby
def key
  @key || @key = Key.find_by_key(params[:key])
end
```

There are two pieces of knowledge that we need to explain this along with how || works thoroughly: 

> "Undefined instance variable returns nil" ([source](https://www.rubyguides.com/2019/07/ruby-instance-variables/))

> "a ||= b approximatively translates to a || a = b and not a = a || b" ([source](https://www.ruby-forum.com/t/the-definitive-list-of-or-equal-threads-and-pages/136446))

What happens when `key` method is called first:
- `@key` is undefined => thus its value is `nil`
- `nil || @key = Key.find_by_key...` will return `@key = Key.find_by_key...` as the first argument is falsy
- => `@key` will have the value returned by `Key.find_by_key(params[:key])`

Now let's run this a second time. What happens then? 

It depends on the previous result of`Key.find_by_key(params[:key])`. 

If a Key was found, then the initial expression will be evaluated as `Truthy || Object` and return the Truthy object that in this case is `@key`:

```ruby
@key || @key =  Key.find_by_key(params[:key])

# when @key is not falsy is returning:

@key
```
If a Key was not found and the result of `find_by_key` is `nil` then the second run will be evaluates as `Falsy || Object` and return the Object:

```ruby
@key || @key = Key.find_by_key(params[:key])

# when @key is falsy is returning and executing the initialize of @key:

@key = Key.find_by_key(params[:key])
```


# More examples: 

## Transforming `true` in 1 and `false` in 0

```ruby
# This method transforms true to 1 and false to 0
def transform(x)
x && 1 || 0
end
```
([source](https://stackoverflow.com/questions/13537206/how-do-i-convert-boolean-values-to-integers) for the idea to convert boolean to integers using short-circuit operators)

When x is true: 

`true && 1 || 0` => `(true && 1) || 0` => `1 || 0`  =>  `1`

When x is false:

`false && 1 || 0` => `(false && 1) || 0` => `false || 0` => 0`

## Responding with one object or another if argument is true

```ruby
def success(response)
  puts "Success"
end

def error
  puts "Error"
end

def render(response)
   response && success(response) || error
end

render(nil) #=> "Error"
render(false) #=> "Error"
render(true) # => "Success"
render(1) #=> "Success"
```

