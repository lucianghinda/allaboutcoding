---
title: "About Ruby: A tale of searching for the main"
seoTitle: "Ruby's Journey: Quest for a method defined in main"
seoDescription: "Explore the question: Where is a method defined in Ruby when not defined inside an object?"
datePublished: Wed Oct 18 2023 03:14:12 GMT+0000 (Coordinated Universal Time)
cuid: clnv6ixud00080ame89i4bric
slug: about-ruby-a-tale-of-searching-for-the-main
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1697597762162/c7ecfdd2-4dc6-4b53-9e95-13dd55407ca4.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1697598756135/3c9a2ec5-6482-44fc-9097-8a1e5efa781e.png
tags: programming-blogs, ruby, ruby-on-rails, programming-languages

---

Here is a fun way to play with Ruby while trying to explore some basic concepts.

If you are learning Ruby, I hope this article demonstrates that Ruby enables excellent self-exploration, and most questions about its functionality can be answered through hands-on experimentation and just a little documentation.

## Context

Someone asked me the following question:

> **When I write in IRB or in a file the following**

```ruby
def my_method
end
```

> Where is the `my_method` created? What object will contain this method? And what is the access modifier for this method?

## Detective Tools

I will use some simple but powerful tools:

1. Ruby interpreter installed on the local machine
    
2. Ruby documentation at [https://docs.ruby-lang.org/en/3.2](https://docs.ruby-lang.org/en/3.2/)
    

## Solution

Instead of giving a direct answer, let's put on the hat of an investigator and try to find some clues about this while having fun with Ruby.

### First question: When I run a script who is there already?

First, let's create a file called `exploration.rb` and ask it about `self`:

```ruby
# exploring.rb
puts "Who is here? #{self}"

# => `Who is here? main`
```

We notice this return `main`. But what is this `main`?

Maybe we can ask Ruby about it. I will add a couple of methods to find out more about `main` while trying to dig around:

```ruby
puts "Who is here: #{self}"
puts "What is your class: #{self.class}"
puts "What are your ancestors: #{self.class.ancestors}"
puts "What is the current exection stack: #{caller.inspect}"

# =>
Who is here: main
What is your class: Object
What are your ancestors: [Object, Kernel, BasicObject]
What is the current caller stack: []
```

We found out that `main` exists inside `Object`. Good, let's poke around then:

```ruby
puts "Is `main` a public method? #{self.public_methods.include?(:main)}"
puts "Is `main` a public method? #{self.protected_methods.include?(:main)}"
puts "Is `main` a private method? #{self.private_methods.include?(:main)}"
puts "Is `main` a singleton method? #{self.singleton_methods.include?(:main)}"

# => 
Is `main` a public method? false
Is `main` a public method? false
Is `main` a private method? false
Is `main` a singleton method? false
```

It appears that `main` is not a method. That seems strange, right? If `main` is not a method then what could it be?

I think we should now explore how we are asking the question: we are using `puts` to ask the question. Here is what [the documentation](https://docs.ruby-lang.org/en/3.2/IO.html#method-i-puts) says

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697512187685/fa5a696d-0ae5-46ce-9c25-8967e646fb68.png align="center")](https://docs.ruby-lang.org/en/3.2/IO.html#method-i-puts)

Then maybe something is deturning the `puts self` and that something could be a method `to_s` on `Object`. Here let's check a bit the documentation for [Object#to\_s](https://docs.ruby-lang.org/en/3.2/Object.html#method-i-to_s)

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697095009683/c71beafd-ee66-4419-9303-65f319153654.png align="center")](https://docs.ruby-lang.org/en/3.2/Object.html#method-i-to_s)

**CLUE A - FOUND -&gt;** `main` is just a string printed by Ruby in the initial execution context. Nothing special here.

### Second: Where is a method created when I don't define an object?

For this, we will create a second file called `exploration_method.rb` and again ask it about itself:

```ruby
def my_method
  puts "Who is here: #{self}"
  puts "What is your class: #{self.class}"
  puts "What are your ancestors: #{self.class.ancestors}"
  puts "What is the current exection stack: #{caller.inspect}"
  puts "What is your current file: #{__FILE__}"
end

my_method

# => 
Who is here: main
What is your class: Object
What are your ancestors: [Object, Kernel, BasicObject]
What is the current exection stack: ["exploration_method.rb:10:in `<main>'"]
What is your current file: exploration_method.rb
```

**CLUE B - FOUND -&gt;** `my_method` is created inside `Object`

### Third: What kind of method is `my_method`?

Let's ask `Object` about this method:

```ruby
def my_method
end

puts "Is `my_method` a public method? #{Object.public_methods.include?(:my_method)}"
puts "Is `my_method` a private method? #{Object.private_methods.include?(:my_method)}"
puts "Is `my_method` a private method? #{Object.protected_methods.include?(:my_method)}"
puts "Is `my_method` a singleton method? #{Object.singleton_methods.include?(:my_method)}"

# => 
Is `my_method` a public method? false
Is `my_method` a private method? true
Is `my_method` a private method? false
Is `my_method` a singleton method? false
```

**CLUE C - FOUND** -&gt; Seems like `my_method` is a private method defined on `Object`

I will try to test this. I know that `public_send` can only call public methods on an object. And I know that `send` will call any method.

I will create a simple method that will print `self` and `object_id` and try out various things:

```ruby
def my_method
  puts "Who is here: #{self} with object_id #{self.object_id}"
end

my_method

# => 
Who is here: main with object_id 60
```

What happens if I try the following things:

```ruby
def my_method
  puts "Who is here: #{self} with object_id #{self.object_id}"
end

Object.new.my_method

# => 
testing.rb:5:in `<main>': private method `my_method' called for #<Object:0x00000001054229c0> (NoMethodError)
```

What about using `public_send` -&gt; the same result

```ruby
def my_method
  puts "Who is here: #{self} with object_id #{self.object_id}"
end

Object.new.public_send(:my_method)

# => 
testing.rb:5:in `public_send': private method `my_method' called for #<Object:0x0000000105952968> (NoMethodError)
```

Then `send` should work:

```ruby
def my_method
  puts "Who is here: #{self} with object_id #{self.object_id}"
end

Object.new.send(:my_method)

# => 
Who is here: #<Object:0x0000000107412960> with object_id 60
```

So if `my_method` will be defined on the `Object` then I must be able to use it in my custom objects right?

```ruby
def my_method
  puts "Who is here: #{self} with object_id #{self.object_id}"
end

class SimpleObject
  def test
    my_method
  end
end

SimpleObject.new.test
```

Indeed it works! I now confirmed in two ways that `my_method` is indeed created at runtime in `Object`

### Four: Is `my_method` defined in Object or added to Object?

Just to push things more, let's try to find out what kind of mechanism is used to add this method inside the `Object` (well from how I formulated this you probably can guess):

```ruby

def my_method
  puts "Who is here: #{self} with object_id #{self.object_id}"
end

class SimpleObject
  def test
    my_method
  end
end

puts "Are you defining `my_method`? #{SimpleObject.private_method_defined?(:my_method)}"
uts "Do you have this method? #{Object.private_methods.include?(:my_method)}"

# => 
Are you defining `my_method`? true
Do you have this method? true
```

This is a method that was added to the `Object` and `Object` is defining this method as a private method.

We can test this by using [`Object#method_added`](https://docs.ruby-lang.org/en/3.2/Module.html#method-i-method_added)

```ruby
class Object
  def self.method_added(method_name)
    puts "I just added #{method_name.inspect} on #{self}"
  end
end

def my_method
  puts "Who is here: #{self} with object_id #{self.object_id}"
end

# => 
I just added :my_method on Object
```

**CLUE D - FOUND** The method is *defined* on `Object` but as a private method. That's why it is accessible everywhere, but that is also why if you add this kind of methods, you would be polluting every object with extra private methods if you define top-level methods like that or you might redefine already existing methods.

## Conclusion

In conclusion, when defining a method without specifying an object in Ruby, the method is created as a private method inside the `Object` class.

This method can be called on any custom object, as it is added during the execution of the file.

Make sure you are not defining in the main context a method that is already defined by Object, Kernel or any other classes by Ruby cause that will create some not-wanted side effects.

Check out the result of executing the following code that I added inside a file called `side_effects.rb`:

```ruby
class SimpleObject
  def my_method
    puts "Who is here inside? #{self}"
    puts "Where is `to_s` defined? #{method(:to_s).source_location.inspect}"
  end
end

puts "Who is here in the initial execution context? #{self}"
SimpleObject.new.my_method

def to_s
  "THIS IS NOT MAIN"
end

class SecondSimpleObject
  def my_method
    puts "Who is here inside? #{self}"
    puts "Where is `to_s` defined? #{method(:to_s).source_location.inspect}"
  end
end

puts "Who is here in the initial execution context? #{self}"
SecondSimpleObject.new.my_method
```

The output will be:

```ruby
Who is here in the initial execution context? main
Who is here inside? #<SimpleObject:0x00000001057b21f8>
Where is `to_s` defined? nil
Who is here in the initial execution context? main
Who is here inside? THIS IS NOT MAIN
Where is `to_s` defined? ["side_effects.rb", 12]
```

Notice that `to_s` from SecondSimpleObject has now the value `THIS IS NOT MAIN` and it is reported to be defined in `side_effects.rb` file?

## Updates

* [Ufuk Kayserilioglu](https://ufuk.dev) gave me [valuable feedback](https://ruby.social/@ufuk/111256244982093392) on an earlier version of this article that I incorrectly used `method_defined?` instead of `private_method_defined?` in the section about if method was added or defined. Thus I refactored that section and now it states clearly that `my_method` is defined as a private method on the `Object`
    

---

Enjoyed this article?

Join my [**Short Ruby News**](https://shortruby.com/) **newsletter** for weekly Ruby updates from the community. For more Ruby **learning resources**, visit [**rubyandrails.info**](http://rubyandrails.info)**.** You can also find me [**on**](http://onruby.social/) [**Ruby.social**](http://Ruby.social) **or** [**Linkedin**](https://linkedin.com/in/lucianghinda) **or** [**Twitter**](https://x.com/lucianghinda)