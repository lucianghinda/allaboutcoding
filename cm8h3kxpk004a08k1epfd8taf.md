---
title: "How to create value objects in Ruby - the idiomatic way"
seoTitle: "Creating Ruby Value Objects: The Idiomatic way"
seoDescription: "Learn to create value objects in Ruby using idiomatic practices, focusing on immutability, comparability, and the modern `Data` class"
datePublished: Thu Mar 20 2025 08:36:49 GMT+0000 (Coordinated Universal Time)
cuid: cm8h3kxpk004a08k1epfd8taf
slug: how-to-create-value-objects-in-ruby-the-idiomatic-way
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1742459187463/2aac11ab-745f-4302-83ee-79d7949614db.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1742459200982/22875328-49e0-409b-a1b9-4b1d2986bed5.png
tags: programming-blogs, ruby, ruby-on-rails, best-practices

---

When writing Ruby OOP, a typical pattern might be to create an object to group multiple values together meaningfully and sometimes also add some extra methods (computed properties, predicates, representations, …) to allow the object to respond to various situations.

Here is an exploration of how to create value-alike objects in Ruby and what I think is the modern idiomatic way.

## What is a value-alike object?

If you want to read an article about this concept, I recommend [Value Object](https://martinfowler.com/bliki/ValueObject.html) by Martin Fowler. He explains this concept very well with examples and references. I invite you to read that article. It is not that long.

They are simple objects that have the following properties:

* Comparable by type and value (that means two objects having the same values as attributes and being the same class will be equal)
    
* Immutable (once set their attributes, they should not be allowed to be changed)
    

The concept is useful when you have to carry around multiple related values, and you need them together in most cases. When talking about Ruby and Ruby OOP, a value object is a simple immutable object that represents a concept in your domain and knows to respond to some simple messages like: what is the value of this property, what is your representation as a string, are you equal to this object in value and type and so on.

The simplest example is a `Point` that in 2D has `x` and `y` properties.

Thus, instead of carrying on a hash `{ x: 20, y: 30 }`, you carry on an object: `Point.new(x: 20, y:20)` that you can interrogate and ask questions. For example, you might want to know in which quadrant the point is, and making this an object allows you to add methods to it, like `quadrant`

Or I can ask `distance_to(another_point)`, and it will tell me something along the lines of `Math.sqrt((x - other.x) ** 2 + (y - other.y) ** 2)`

But most of the time, you need to ask that object `point.x` or `point.y` and have a good name for both the class and the attributes.

## Why use a value object?

You might ask yourself, why use a value object?

Here are some arguments for using a value object:

1. Keep data together using an immutable object so you can be sure that as you pass an instance of this object through your code, nothing has changed it.
    
2. It is a group of values that should be together under a meaningful name
    
3. It is greppable, meaning you can easily find all the places in your codebase where this object is used, making it simple to refactor when needed.
    

When comparing a value object with using a more base structure like a Hash, the advantages over Hash are:

* Naming: the structure of the values together have a name
    
* Greppability: because now this structure has a name, it is easier to find it in the codebase
    
* Flexibility: you can add utility methods, computed properties, or predicates
    

## The Idiomatic Ruby way since 2022

The idiomatic way to do this in Ruby should be to [use `Data` class](https://docs.ruby-lang.org/en/master/Data.html). Here is what Ruby's official documentation says about it:

> Class Data provides a convenient way to define simple classes for value-alike objects.

[![Screenshot of the definition of the Data object in Ruby](https://cdn.hashnode.com/res/hashnode/image/upload/v1742371331135/4d55f78a-e883-4d1a-b99c-8ab65ff0589e.png align="center")](https://docs.ruby-lang.org/en/master/Data.html)

We must thank [Victor Shepelev](https://zverok.space) for proposing this structure in Ruby and continuing the discussion about it until it was accepted and implemented.

### How to define and create an instance of a Data class

Say you want to pass along an object representing an amount and save the value and the currency.

You can define this like:

```ruby
Price = Data.define(:amount, :currency)
```

There are multiple ways to instantiate an objects from a Data class:

**Using required keyword arguments**

```ruby
price = Price.new(amount: 50, currency: "USD")
# => #<data Price amount=50, currency="USD">
```

**Using positional arguments**

```ruby
price = Price.new(50, "USD")
# => #<data Price amount=50, currency="USD">
```

**Using alternative form construct**

```ruby
price = Price["50", "USD"]
# => #<data Price amount="50", currency="USD">
```

**Using alternative construction hash-like form**

```ruby
price = Price[amount: "50", currency: "USD"]
# => #<data Price amount="50", currency="USD">
```

### Properties of the `Data` object in Ruby

A Data object has two important properties in my view:

1. **It is immutable** so there are no setters defined and you cannot define setters
    

```ruby
# You cannot change the value

price = Price.new(amount: 50, currency: "USD")

# The following will raise an exception
price.amount = 50 
# => `<main>': undefined method `amount=' for #<data Price amount=50, currency="USD"> (NoMethodError)
```

Even if you try to define a setter, it will not work:

```ruby

Price = Data.define(:amount, :currency) do
    def value=(new_value)
      @value = new_value
    end
end

price = Price.new(amount: 50, currency: "USD")
price.amount = 100 # This will return an exception
# => in 'Price#amount=': can't modify frozen #<Class:#<Price:0x00000001259166a0>>: #<data Price amount=20, currency="USD"> (FrozenError)
```

2. **It is comparable by type and value**
    

```ruby
Price = Data.define(:amount, :currency)

price1 = Price.new(amount: 50, currency: "USD")

price2 = Price.new(amount: 40, currency: "USD")

price3 = Price.new(amount: 50, currency: "USD")

# While these are all different objects
puts price1.object_id
puts price2.object_id
puts price3.object_id

# You will notice that price1 and price3 will be equal
price1 == price3 # true
price1 == price2 # false
```

It has some other properties but I think these two are the main ones that we should think about when using Data as a value object.

### Changing one or many values

The only way to achieve this is to use `with` and create a new value object:

```ruby
price_a = Price.new(amount: 50, currency: "USD")

price_b = price_a.with(amount: 100)
# => #<data Price amount=100, currency="USD">
```

### Defining custom methods

You might notice that if you browse [the documentation](https://docs.ruby-lang.org/en/master/Data.html) that the Data object has a limited number of methods: there are 10 methods specific to this object (not taking into consideration of course methods inherited from its ancestors: `[Object, Kernel, BasicObject]`)

But you can define custom computed properties on it if you want to by passing a block to `define`:

```ruby
Price = Data.define(:amount, :currency) do
    def to_s 
        "#{currency} #{amount}"
    end
end

price = Price.new(amount: 50, currency: "USD")
price.to_s # => "USD 50"
```

### Default values using an initializer

In case you want to have default values, you can override the initializer like this:

```ruby
Price = Data.define(:amount, :currency) do
    def initialize(amount:, currency: "USD") 
        super
    end
end

# with the shortest form possible, like
Price = Data.define(:amount, :currency) do
    def initialize(amount:, currency: "USD") = super
end
```

### Checking the kind (or type) of the attributes

While this is not a feature of Data object, I feel like it is working nice along with it: you can check for type (or, better said, kind of the arguments) using Pattern Matching:

```ruby
Price = Data.define(:amount, :currency) do
    def initialize(amount:, currency:)
        amount => Integer
        currency => String
        
        super
    end
end
```

This can be useful if you, for example want to define Currency as:

```ruby
Currency = Data.define(:code, :name, :symbol, :decimal)
USD = Currency.new(code: "USD", name: "US Dollar", symbol: "$", decimal: 2)
EUR = Currency.new(code: "EUR", name: "Euro", symbol: "€", decimal: 2)

Price = Data.define(:amount, :currency) do
    def initialize(amount:, currency:)
        amount => Integer
        currency => Currency
        
        super
    end
end

# ✅
p1 = Price.new(amount: 10, currency: USD) 
# => #<data Price amount=10, currency=#<data Currency code="USD", name="US Dollar", symbol="$", decimal=2>>

# ❌
p2 = Price.new(amount: 10, currency: "USD")
# => in `initialize': "USD": Currency === "USD" does not return true (NoMatchingPatternError)
```

## Data class vs Struct

You can use Struct just as well as using the Data class but it will not create immutable objects. You can of course freeze the instances but the Data object gives that for free.

In case you want to create a Struct here is how it might look like:

```ruby
Price = Struct.new(:amount, :currency, keyword_init: true)

price = Price.new(amount: 50, currency: "USD")
# => <struct Price amount=50, currency="USD">
```

It supports also providing a block when creating the structure to add extra methods as we did it for the Data.

## A bit on the immutability

Please note that the Data objects are frozen but not deep frozen. That means that if you put inside a Data object attribute a structure that is not immutable then you can change those properties.

**My advice is to combine Data objects only with Data objects or immutable or immediate objects to make sure that everything remains immutable.**

I recommend you to do this:

```ruby
Currency = Data.define(:code, :name, :symbol, :decimal)
Price = Data.define(:amount, :currency)

EUR = Currency.new(code: "EUR", name: "EURO", symbol: "€", decimal: 2)
price = Price.new(amount: 50, currency: EUR)

price.currency.code = "MyEUR"
# undefined method 'code=' for an instance of Currency (NoMethodError)
```

While if you try to put there an object that can be changed like say a Struct it will allow you to change it:

```ruby
Currency = Struct.new(:code, :name, :symbol, :decimal, keyword_init: true)
Price = Data.define(:amount, :currency)

EUR = Currency.new(code: "EUR", name: "EURO", symbol: "€", decimal: 2)
price = Price.new(amount: 50, currency: EUR)
# => #<data Price amount=50, currency=#<struct Currency code="EUR", name="EURO", symbol="€", decimal=2>>

price.currency.code = "MyEUR"
price
# => #<data Price amount=50, currency=#<struct Currency code="MyEUR", name="EURO", symbol="€", decimal=2>>
```

The same goes for putting inside a hash:

```ruby
Price = Data.define(:amount, :currency)
EUR = { code: "EUR", name: "EURO", symbol: "€", decimal: 2 }

price = Price.new(amount: 50, currency: EUR)
# => #<data Price amount=50, currency={code: "EUR", name: "EURO", symbol: "€", decimal: 2}>
price.currency[:code] = "MyEUR"
price
# => #<data Price amount=50, currency={code: "MyEUR", name: "EURO", symbol: "€", decimal: 2}>
```

So you need to take care of freezing the objects that you put inside yourself if you want them to remain unchanged.

### **Trick: You can use Ractor shareable to deep freeze a Data object**

This is probably not the purpose of the Ractor.make\_shareable method, but it is a nice trick to try:

```ruby
Currency = Struct.new(:code, :name, :symbol, :decimal, keyword_init: true)
Price = Data.define(:amount, :currency)

EUR = Currency.new(code: "EUR", name: "EURO", symbol: "€", decimal: 2)
price = Ractor.make_shareable(Price.new(amount: 50, currency: EUR))
# => #<data Price amount=50, 
# => currency=#<struct Currency code="EUR", name="EURO", symbol="€", decimal=2>>

price.currency.code = "MyEUR"
# => in '<main>': can't modify frozen 
# => Currency: #<struct Currency code="EUR", name="EURO", symbol="€", decimal=2> (FrozenError)
```

## Some conclusions

I think the Data is a great addition to the Ruby language and I try to reach out to it everytime I need to create a value object. I like the immutability and the comparison by type and value that is created by default.

It also defines `deconstruct` and `deconstruct_keys` if you want to use the Data object in pattern matching (Struct defines them too in case you want to use Struct).

If you want to read more about this I recommend at least two resources:

* The feature request on the Ruby Bug Tracker: [https://bugs.ruby-lang.org/issues/16122](https://bugs.ruby-lang.org/issues/16122)
    
* The great documentation about it maintain by Victor Shepelev here [https://rubyreferences.github.io/rubychanges/3.2.html#data-new-immutable-value-object-class](https://rubyreferences.github.io/rubychanges/3.2.html#data-new-immutable-value-object-class)
    

---

If you like this article:

👐 Interested in learning how to improve your developer testing skills? Join my live online workshop about [**goodenoughtesting.com**](http://goodenoughtesting.com/) **\- to learn test design techniques for writing effective tests**

👉 Join my [**Short Ruby Newsletter**](https://newsletter.shortruby.com/) for weekly Ruby updates from the community and visit [**rubyandrails.info**](http://rubyandrails.info/)**, a directory with learning content about Ruby.**

🤝 Let's connect on [**Bluesky**](https://bsky.app/profile/lucianghinda.com), [**Ruby.social**](http://ruby.social/)**,** [**Linkedin**](https://linkedin.com/in/lucianghinda)**,** [**Twitter**](https://x.com/lucianghinda) **where I post mostly about Ruby and Ruby on Rails.**

🎥 Follow me on [**my YouTube channel**](https://www.youtube.com/@shortruby) for short videos about Ruby/Rails