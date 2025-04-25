---
title: "Prefer getter methods over instance variables inside Ruby objects"
seoTitle: "Use getter methods instead of instance variables in Ruby objects"
seoDescription: "Use Ruby getter methods over instance variables for clearer errors and easier debugging"
datePublished: Fri Apr 25 2025 04:39:06 GMT+0000 (Coordinated Universal Time)
cuid: cm9waxw3e000j09jjhfxe0rjx
slug: prefer-getter-methods-over-instance-variables-inside-ruby-objects
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1745555841619/38d5afab-f529-4010-a03e-f4c5d2e31b06.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1745555856690/5e922ce4-5609-43cb-8474-2819373cca7c.png
tags: design-patterns, ruby, ruby-on-rails, properties, design-principles

---

I prefer to use getter methods instead of instance variables inside Ruby classes. I will compare getters to instance variables, focusing mainly on `attr_reader`.

*Note:* The code in this article was tested on **Ruby 3.4.1**

## Defining getters to access instance variables

In most codebases that I saw, the primary use of an `attr_reader` is to provide public access to an instance variable by defining an `attr_reader` like this:

```ruby
class Link
  attr_reader :url 
  
  def initialize(url)
    @url = url
  end  
end
```

Funny thing you can define getters using string if you want to and Ruby will convert it to symbol and define the same method:

```ruby
class Link
  attr_reader 'url'
  
  def initialize(url)
    @url = url
  end  
end

link = Link.new("https://shortruby.com")
puts link.methods.include?(:url) # true
```

You can also use `attr` to define a getter:

```ruby
class Link
  attr :url 
  
  def initialize(url)
    @url = url
  end  
end
link = Link.new("https://shortruby.com")
puts link.methods.include?(:url) # true
puts link.methods.include?(:url=) # false
```

There was also the option to use `attr :url, true` to define a setter but that is deprecated. Also doing `attr :url, false` is deprecated. So only `attr :url` remains which is equivalent to `attr_reader :url`

**I think using** `attr_reader` **is better because the name of the method describes what is does.**

### You can define a private getter

If you want to you can define a private getter like this:

```ruby
class Link
  def initialize(url)
    @url = url
  end  
  
  private 
  
  attr_reader :url 
end
```

And my plan in this article is to try to show why you should do this and what advantages it might bring you. I am not saying you should do this all the time, but I think it could be a case of using getters also inside the object where you are defining it.

## A short intermission about private and public methods

When I think about a Ruby object I try to minimize the public methods that it exposes. My line of thinking is that all methods should be private unless there is a real reason to make that method public.

When making a method public you are signing a contract with another developer that you will keep that method the same for as long as possible (the same parameters, the same return, the same effects …). So I think the default position should be to limit your liabilities.

A getter is a method so when you think about defining getters you should think if you really need to expose them and make them public.

## Accessing undefined instance variables

As my main assertion here is to use getters when accesing instance variables inside the current object, I think it is important to talk a bit about instance variables and what is their value when they are not created:

```ruby
class Simple
  def is_it_defined?
    instance_variable_defined?(:@this_does_not_exists)
  end

  def value
    @this_does_not_exists
  end
end

simple = Simple.new
puts simple.is_it_defined? # => false
puts simple.value.inspect # => nil
```

We know that if an instance variable is not initialized/defined but it is directly accessed it will return `nil` but will not throw an error.

This opens the case of some bugs related to checking truthiness:

```ruby
class Simple
  def initialize(payload)
    @payload = payload
  end
  
  def run
    if @paylod
      puts "Run with payload"
    else 
      puts "Without payload"
    end
  end
end
```

Notice I wrote `@paylod` instead of `@payload` and so the result will be:

```ruby
Simple.new("Something").run

# Got => Without payload ❌

# Expected => Run with payload
```

Let’s try a second example, this one a bit more complex:

```ruby
class InvoiceBuilder
  PREFIXES_FOR_EUR = ["INV", "I"]

  def initialize(prefix)
    @prefix = prefix
  end

  def currency
    return "EUR" if PREFIXES_FOR_EUR.include?(@prefix)

    "USD" 
  end
end
```

If we run two tests it will pass:

```ruby
invoice_number_builder = InvoiceBuilder.new("INV")
puts invoice_number_builder.currency
# => EUR 

invoice_number_builder = InvoiceBuilder.new("CUSTOM")
puts invoice_number_builder.currency
# => USD
```

Again what if there will be a typo (notice the variable `@prefx` in the `currency` method:

```ruby
class InvoiceBuilder
  PREFIXES_FOR_EUR = ["INV", "I"]

  def initialize(prefix)
    @prefix = prefix
  end

  def currency
    return "EUR" if PREFIXES_FOR_EUR.include?(@prefx)

    "USD" 
  end
end
```

The same tests will fail:

```ruby
invoice_number_builder = InvoiceBuilder.new("INV")
puts invoice_number_builder.currency
# => USD ❌

invoice_number_builder = InvoiceBuilder.new("CUSTOM")
puts invoice_number_builder.currency
# => USD
```

## The case for using a getter

Even if you don’t plan to expose the getter as a public interface you can define a getter and use it inside the object:

```ruby
class InvoiceBuilder
  PREFIXES_FOR_EUR = ["INV", "I"]

  def initialize(prefix)
    @prefix = prefix
  end

  def currency
    return "EUR" if PREFIXES_FOR_EUR.include?(prefix)

    "USD"
  end
  
  private 
  
  attr_reader :prefix
end
```

And in case there is a typo like the following one:

```ruby
class InvoiceBuilder
  PREFIXES_FOR_EUR = ["INV", "I"]

  def initialize(prefix)
    @prefix = prefix
  end

  def currency
    return "EUR" if PREFIXES_FOR_EUR.include?(prefx)

    "USD"
  end
  
  private 
  
  attr_reader :prefix
end
```

When running the following code:

```ruby
invoice_builder = InvoiceBuilder.new("INV")
puts invoice_builder.currency
```

We will get a very clear error:

```ruby
test-with-attr-reader.rb:9:in 'InvoiceBuilder#currency': 
undefined local variable or method 'prefx' for an instance of InvoiceBuilder (NameError)

    return "EUR" if PREFIXES_FOR_EUR.include?(prefx)
                                              ^^^^^
 Did you mean?  prefix
               @prefix
        from test-with-attr-reader.rb:20:in '<main>'
```

Notice the difference:

* In the version with instance variable the execution silently returned an apparent valid response
    
* In the version with the getter the execution will raise an exception and will also tell us exactly where the typo is located.
    

## Protecting against mistakes with tests

You can of course protect against these kind of mistakes by writing some functional tests like the following ones (I am using here Minitest but the test framework does not make any difference).

```ruby
class InvoiceBuilderTest < Minitest::Test
  def test_currency_returns_eur_for_known_prefixes
    prefix = "INV"
    builder = InvoiceBuilder.new(prefix)
    
    assert_equal "EUR", builder.currency
  end

  def test_currency_returns_usd_for_unknown_prefixes
    prefix = "CUSTOM"
    builder = InvoiceBuilder.new(prefix)
    
    assert_equal "USD", builder.currency
  end
end
```

And if there is a typo in case of the version with the instrance variable if we run it we will get:

```ruby
# Running:
.F
Finished in 0.000339s, 5899.7052 runs/s, 5899.7052 assertions/s.

  1) Failure:
InvoiceBuilderTest#test_currency_returns_eur_for_known_prefixes [instance_variable_with_typo.rb:30]:
Expected: "EUR"
  Actual: "USD"

2 runs, 2 assertions, 1 failures, 0 errors, 0 skips
```

### How to use this in Rails

When talking about Rails I generally think being close to defaults is the sane choice. Still you can use this technique there too with either a getter or with a local variable instead of an instance variable.

Having an ERB file like this:

```erb
<% if @payment %>
  <p>Payment exists!</p>
<% else %>
  <p>No payment found.</p>
<% end %>
```

You can rewrite it like this:

```erb
<% if payment %>
  <p>Payment exists!</p>
<% else %>
  <p>No payment found.</p>
<% end %>
```

And then you can pass in the controller the `payment` using `locals`:

```ruby
class PaymentsController < ActionController::Base
  include Rails.application.routes.url_helpers

  def show
    @payment = Object.new

    render locals: { payment: payment }
  end

  private

    attr_reader :payment
end
```

In case there is for example a typo in ERB:

```erb
<% if paymnt %>
  <p>Payment exists!</p>
<% else %>
  <p>No payment found.</p>
<% end %>
```

When you run your tests (or the web app and access that page) you will get an error that will look like this:

```bash
ActionView::Template::Error (undefined local variable or method 'paymnt' for an instance of #<Class:0x0000000121ad6228>)
Caused by: NameError (undefined local variable or method 'paymnt' for an instance of #<Class:0x0000000121ad6228>)

Information for: ActionView::Template::Error (undefined local variable or method 'paymnt' for an instance of #<Class:0x0000000121ad6228>):
    1: <% if paymnt %>
    2:   <p>Payment exists!</p>
    3: <% else %>
    4:   <p>No payment found.</p>

app/views/payments/show.html.erb:1
app/controllers/payments_controller.rb:5:in 'PaymentsController#show'
```

Of course when doing this in a Rails controller you can get the benefit of having this nice error that tells you where you are trying to use a method that does not exists by sending local variables from the controller:

```ruby
class PaymentsController < ApplicationController
  def show
    payment = Object.new

    render locals: { payment: payment }
  end
end
```

This is much more simpler than writing an instance variable.

In the end the change that you have to explicitly call `render` with `locals` if you are not already calling it and pass it local variables instead of setting an instance variable.

```diff
def show
- @payment = Object.new
+ payment = Object.new
+ render locals: { payment: payment }
end
```

One note before we move on: This is not the default Rails way so before start doing this think carefully if you plan to adopt this on your entire codebase. The

## Why prefer methods instead of instance variables?

Summarising the main reasons that I see for prefering method calls instead of instance variables (or in case of the Rails example local variables with render locals):

1. Fails fast because it throws an exception
    
2. Ruby will point to the exact line of code that failed so identifying the root cause of the exception is easier
    

If you're worried about performance, in most cases, the difference isn't significant. Here is [a benchmark](https://gist.github.com/lucianghinda/3264a3bea89aa40613b06815552d0771) I run on my laptop (M3 Pro 32GB):

```ruby
Simple time benchmark (lower is better):
                               user     system      total        real
Instance variable access:  0.024115   0.000012   0.024127 (  0.024128)
Getter access:             0.027965   0.000010   0.027975 (  0.027979)

Iterations per second (higher is better):
ruby 3.4.1 (2024-12-25 revision 48d4efcb85) +PRISM [arm64-darwin23]
Warming up --------------------------------------
Instance variable access:
                         3.571M i/100ms
      Getter access:     3.215M i/100ms
Calculating -------------------------------------
Instance variable access:
                         35.951M (± 1.0%) i/s   (27.82 ns/i) -    182.110M in   5.065990s
      Getter access:     31.687M (± 1.1%) i/s   (31.56 ns/i) -    160.749M in   5.073622s

Comparison:
Instance variable access:: 35951125.5 i/s
      Getter access:: 31687428.0 i/s - 1.13x  slower
```

## You are already doing this in some cases

I want to assert that this change is not so big. Because you are already doing this in a couple of cases:

1. Predicates
    
2. Memoization and Lazy Initiatialization
    

For example when having an instance variable that is used as a boolean we mostly write a method to make it a predicate. See in this example the `vat_predicate?` method:

```ruby
class InvoiceBuilder
  VAT_PERCENTAGE = 0.19
  
  def initialize(total:, vat_included:)
    @total = total
    @vat_included = vat_included
  end

  def amount
    return total unless vat_included? 
    
    (total - (total * VAT_PERCENTAGE)).truncate(2)
  end
  
  private 
  
    attr_reader :total
    
    def vat_included?
      @vat_included
    end
end
```

In case of memoization we do something like this (see the `products` method):

```ruby
class Invoice  
  def initialize(product_ids:)
    @product_ids = product_ids
  end

  def total       = products.sum { _1.price } 
  def print_items = products.each { "#{it.name} | #{it.unit} | #{it.quantity}" }
  
  private 
  
    attr_reader :product_ids
    
    def products 
      @products ||= Product.where(id: product_ids).order(name: :asc)
    end
end
```

## Some conclusion

The typing mistakes I presented here are easy to spot because the examples are very simple. I think this is how code should look like: simple, with few lines of code if possible. If you have code like this there is no need to adopt this technique becuase you will be able to spot a mistake very easily.

But generally, code can be more complex, making mistakes harder to find. You'll need tests to catch cases like checking truthiness against nil values. Be aware of developer confirmation bias when writing tests, as this can make tests appear correct when they're not. Protect against these issues by setting Ruby to raise an exception. This aligns with the shift-left philosophy of catching bugs early, giving you quick and clear feedback on these typing mistakes.

---

If you like this article:

👐 Interested in learning how to improve your developer testing skills? Join my live online workshop about [**goodenoughtesting.com**](http://goodenoughtesting.com/) **\- to learn test design techniques for writing effective tests**

👉 Join my [**Short Ruby Newsletter**](https://newsletter.shortruby.com/) for weekly Ruby updates from the community and visit [**rubyandrails.info**](http://rubyandrails.info/)**, a directory with learning content about Ruby.**

🤝 Let's connect on [**Bluesky**](https://bsky.app/profile/lucianghinda.com), [**Ruby.social**](http://ruby.social/)**,** [**Linkedin**](https://linkedin.com/in/lucianghinda)**,** [**Twitter**](https://x.com/lucianghinda) **where I post mostly about Ruby and Ruby on Rails.**

🎥 Follow me on [**my YouTube channel**](https://www.youtube.com/@shortruby) for short videos about Ruby/Rails