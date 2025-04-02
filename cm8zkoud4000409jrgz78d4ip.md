---
title: "More about how to create a Data class in Ruby"
seoTitle: "Creating a Data Class in Ruby: A small guide"
seoDescription: "Learn to create Ruby Data classes using block syntax and inheritance, and understand their differences and use cases"
datePublished: Wed Apr 02 2025 06:55:36 GMT+0000 (Coordinated Universal Time)
cuid: cm8zkoud4000409jrgz78d4ip
slug: more-about-how-to-create-a-data-class-in-ruby
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1743565446560/014dfc08-c712-475f-ab4d-c0b1bc0b83ab.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1743565477111/05c8c1b9-09f9-4c08-baea-60340b87c3ca.png
tags: programming-blogs, ruby, ruby-on-rails, coding

---

If you have not yet read my previous articles about Data class in Ruby, I invite you to read them first:

* [How to create value objects in Ruby - the idiomatic way](https://allaboutcoding.ghinda.com/how-to-create-value-objects-in-ruby-the-idiomatic-way)
    
* [Example of value objects using Ruby's Data class](https://allaboutcoding.ghinda.com/example-of-value-objects-using-rubys-data-class)
    

Here, I will talk about the two ways you can create Data classes and compare them:

* Using the block
    
* Using the inheritance
    

## How to create a Data class

You can define a new Data class using the block syntax:

```ruby
Response = Data.define(:body, :status)
```

Looking at the [current Ruby docs for Ruby version 3.4](https://docs.ruby-lang.org/en/3.4/Data.html) this seems to be the way to do it or at least the documentation is using this way of creating a new Data class. If I don’t miss anything all examples there are using this syntax.

You can also define a new Data class by using the inheritance

```ruby
class Response < Data.define(:body, :status)
end
```

## The inheritance chain

The main difference between them is that the inheritance syntax is defining an extra anonimous class.

Here are the ancestors for the block syntax:

```ruby
Response = Data.define(:body, :status) do
end
Response.ancestors
# [
#   Response, 
#   Data, 
#   Object, 
#   Kernel, 
#   BasicObject
# ]
```

The ancestors for the inheritance syntax are:

```ruby
class Response < Data.define(:body, :status)
end
Response.ancestors
# [
#   Response, 
#   <Class:0x000000011b98abe0>, 
#   Data, 
#   Object, 
#   Kernel, 
#   BasicObject
# ]
```

Notice there an extra `<Class:0x000000011b98abe0>` in the inheritance chain.

This is not something specific for the Data class. It works the same way for Struct too:

```ruby
Response = Struct.new(:body, :status)
Response.ancestors
# [
#   Response,
#   Struct,
#   Enumerable,
#   Object,
#   Kernel,
#   BasicObject
# ]
```

```ruby
class Response < Struct.new(:body, :status)
end
Response.ancestors
# [
#   Response,
#   #<Class:0x000000011d3eabe8>,
#   Struct,
#   Enumerable,
#   Object,
#   Kernel,
#   BasicObject
# ]
```

## When using block syntax with a variable instead of a constant

If you try to assign the Data define block to a variable you will see that the name for that class is not set:

```ruby
Response = Data.define(:body, :status)
object = Response.new(body: {}, status: 200)
object.inspect
# => "#<data Response body={}, status=200>"

response = Data.define(:body, :status)
object = response.new(body: {}, status: 200)
object.inspect
# => "#<data body={}, status=200>"
```

Notice that when we inspect it when assigning to a variable it does not print the “Response” string, that is because the name is not set in case of `response`:

```ruby
Response = Data.define(:body, :status)

Response.name 
# => "Response"

response = Data.define(:body, :status)

response.name
# => nil
```

Again comparing it with Struct, the same happens for Struct:

```ruby
Response = Struct.new(:body, :status)

Response.name 
# => "Response"

response = Struct.new(:body, :status)

response.name
# => nil
```

## Nested constants

What happens if we try to define some constants inside.

If I write this code:

```ruby
Response = Data.define(:body, :status) do
  RateLimit = Data.define(:limit, :remaining, :retry)
end
```

How do I instantiate a new object for `RateLimit`:

* `Response::RateLimit.new …` ?
    
* `RateLimite.new` ?
    

Let’s try it out:

```ruby
rate_limit = Response::RateLimit.new(limit: 10, :remaining: 4, retry: 1743564235)
# => uninitialized constant Response::RateLimit (NameError)
```

```ruby
rate_limit = Response::RateLimit.new(limit: 10, :remaining: 4, retry: 1743564235)
# #<data RateLimit limit=10, remaining=4, retry=1743564235>
```

This is happening because the block does not introduce a new scope and so the constrant is defined in the outher scope.

Here is another example that does not use the Data:

```ruby
Response = Class.new do
  RateLimit = Class.new do
    RETRIES = 10
  end
end

puts Response::RateLimit::RETRIES
# => uninitialized constant Response::RateLimit (NameError)

puts Response::RETRIES 
# => uninitialized constant Response::RETRIES (NameError)

puts RETRIES 
# => 10
```

This is not happening if you use the inheritance syntax for Data object:

```ruby
class Response < Data.define(:body, :status)
  class RateLimit < Data.define(:limit, :remaining, :retry)
    RETRIES = 10
  end
end

puts Response::RateLimit::RETRIES 
# => 10

puts Response::RETRIES 
# => uninitialized constant Response::RETRIES (NameError)

puts RETRIES 
# => uninitialized constant RETRIES (NameError)
```

## Redefining the initializer

There can be multiple times when you will try to redefine the initializer and set some default values or do some other processing in there.

In general it looks like this:

```ruby
class Response < Data.define(:body, :status)
  def initialize(body: {}, status: 200)
    super
  end
end

response = Response.new(body: { message: 'Hello, world!' }, status: 200)
# => #<data Response body={message: "Hello, world!"}, status=200>
```

In case you will try to write the initializer with positional arguments, then you will find out it does not work:

```ruby
class Response < Data.define(:body, :status)
  def initialize(body = {}, status = 200)
    super
  end
end

response = Response.new(body: { message: 'Hello, world!' }, status: 200)
# => in 'Data#initialize': wrong number of arguments (given 2, expected 0) (ArgumentError)

response = Response.new({ message: 'Hello, world!' }, 200)
# => in 'Data#initialize': wrong number of arguments (given 2, expected 0) (ArgumentError)
```

The [documentation in Ruby master](https://docs.ruby-lang.org/en/master/Data.html) is clear about this behavior:

> Note that `Measure#initialize` always receives keyword arguments, and that mandatory arguments are checked in `initialize`, not in `new`. This can be important for redefining initialize in order to convert arguments or provide defaults.

Just remember to always define the initializer with keyword arguments when using the Data define

## Which option to choose the block or the inheritance?

I am not sure there is a clear answer.

I like the simplicity of the block definition. If you don’t plan to add too many extra methods it is so cool to define it in a single line and no need for an extra `end`

```ruby
Response = Data.define(:body, :status)
```

But once you add an extra method, like some default arguments for the initializer:

```ruby
Response = Data.define(:body, :status) do 
  def initialize(body: {}, status: 200) = super
end
```

Then it will not be a big difference in terms of lines of code needed for defining it versus the inheritance:

```ruby
class Response < Data.define(:body, :status)
  def initialize(body: {}, status: 200) = super
end
```

Still if you care about the number of objects created in your Ruby program, remember that the inheritance is creating an extra anonimous class.

---

If you like this article:

👐 Interested in learning how to improve your developer testing skills? Join my live online workshop about [**goodenoughtesting.com**](http://goodenoughtesting.com/) **\- to learn test design techniques for writing effective tests**

👉 Join my [**Short Ruby Newsletter**](https://newsletter.shortruby.com/) for weekly Ruby updates from the community

🤝 Let's connect on [**Bluesky**](https://bsky.app/profile/lucianghinda.com), [**Ruby.social**](http://ruby.social/)**,** [**Linkedin**](https://linkedin.com/in/lucianghinda)**,** [**Twitter**](https://x.com/lucianghinda) **where I post mostly about Ruby and Ruby on Rails.**

🎥 Follow me on [**my YouTube channel**](https://www.youtube.com/@shortruby) for short videos about Ruby/Rails