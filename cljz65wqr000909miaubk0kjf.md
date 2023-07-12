---
title: "About Ruby: pass by value or pass by reference?"
seoDescription: "Explore Ruby's pass-by-value approach, enabling object modification without altering original references, which is essential for efficient object handling"
datePublished: Wed Jul 12 2023 03:36:19 GMT+0000 (Coordinated Universal Time)
cuid: cljz65wqr000909miaubk0kjf
slug: about-ruby-pass-by-value-or-pass-by-reference
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1688527313372/29dc84e7-001b-4418-bf11-b92563a0c886.png
tags: programming-blogs, ruby, programming-languages, programming-tips

---

Here is a simple code in Ruby:

```ruby
def init_options(options)
  options = { widget: true }
end

def add_default_config(options)
  options[:add_on] ||= true
end
```

Do not focus on the code design. There might be better ways to express this, but I want here to focus on what happens when executing the following statements:

```ruby
options = { plugin: true }

add_default_config(options)
puts options # => {:plugin=>true, :add_on=>true}

init_options(options)
puts options # => {:plugin=>true, :add_on=>true}
```

Notice that after executing `init_options` the content is still the same even if inside there is a statement that would instantiate a new hash.

Or take a look at the following code:

```ruby
options = { widget: true }

options.tap do |opt|
  opt[:widget] = false
end

puts options # => { :widget => false }

new_options = { plugin: true }

new_options.tap do |opt|
  opt = {}
  opt = { plugin: false }
end

puts new_options # => { :plugin => true }
```

Why in this case the `new_options` is not changed after the tap was executed?

To answer this we will need to understand how objects are passed in Ruby: pass by value or pass by reference?

Let's dig in to find out how Ruby works when talking about passing objects.

## A variable is a reference to an object

To understand this I will execute a series of statements and explain step by step how they work:

```ruby
options = {}
puts options.object_id # => 60
```

This will do two things:

* Creates a new Hash object that in this specific case on my machine has the id `60`
    
* Create a new label called `options` and associate that with the newly created object.
    

Then calling `options.object_id` on it will return an integer that identifies uniquely that object during the execution of the program.

Imagine that Ruby is just creating a link between the label `options`) and the object that is associated with that label.

This association might look (visually) like this:

```md
+-------------+-----------+
| label       | object id |
+-------------+-----------+
| options     | 60        |
+-------------+-----------+
```

In this case, we can affirm: *"options points to object with id 60"* or *"options references object with id 60"*.

Now I will add a second variable called `new_options`:

```ruby
new_options = options
puts new_options.object_id === options.object_id # => true
```

This second variable will reference now to the same object as `options`:

```md
+-------------+-----------+
| label        | object id |
+-------------+-----------+
| options     | 60        |
| new_options | 60        |
+-------------+-----------+
```

What do you think will happen if I start adding more hash keys to either `options` or `new_options`?

```ruby
options[:plugin] = true
puts options # => { :plugin =>true }
puts new_options # => { :plugin =>true }

new_options[:widget] = true
puts options # => { :plugin =>true, :widget =>true }
puts new_options # => { :plugin =>true, :widget =>true }

# Are they still the same object?
# Yes, they are.
puts new_options.object_id == options.object_id # => true
```

Because they are both references to the same object, calling a method on any of those variables like for example [`[]=`](https://docs.ruby-lang.org/en/3.2/Hash.html#method-i-5B-5D-3D) will be called on the referenced object (in the specific example the object with object\_id 60).

What happens when I try to instantiate a new object and assign that to one of the existing variables?

```ruby
# What if I try to assign the `options` variable to a new object?
options = Hash.new

puts options.object_id # => 80
puts new_options.object_id # => 60
puts new_options # => { :plugin =>true, :widget =>true }
puts options # => {}

# They are not the same object
puts new_options.object_id == options.object_id # => false
```

In this case the `options` will point to a new object while `new_options` is still pointing to the existing one:

```ruby
+-------------+-----------+
| label       | object id |
+-------------+-----------+
| options     | 80        |
| new_options | 60        |
+-------------+-----------+
```

Here we can draw a couple of lessons:

1. Variables are references to objects, but not the objects
    
2. Assigning a new object to a variable will change it to reference the new object
    

## Passing arguments to methods

When passing an argument to a method we are in a way creating a local variable inside that method that references the object that is passed.

Let's start with a simple example:

```ruby
options = { widget: true }

def add_default_config(options)
  puts "Object has id: #{options.object_id}"
end

add_default_config(options) # => Object has id: 60
```

What happens here is that inside the `add_default_config` method a new variable called `options` is created and it is assigned to the same object that is passed to the method as a reference. This means that the `options` variable inside the method is pointing to the same object as the `options` variable outside the method.

To represent the references we will need to add a new column that is defining the scope of the variable:

```plaintext
+--------------------+------------+-----------+
| lexical scope      | label      | object id |
|--------------------|------------|-----------|
| main               | my_options | 60        |
| add_default_config | options    | 60        |
+--------------------+------------+-----------+
```

This simply says:

* There exists a label called `my_options` that is referencing an object with id `60` in the main scope
    
* There exists a label called `options` that is referencing an object with id `60` in the `add_default_config` scope or the local scope of the method `add_default_config`
    

Now let's try to change the object inside the method:

```ruby
my_options = { widget: true }
puts my_options.object_id # => 60

def add_default_config(options)
  puts "Object has id: #{options.object_id}"
  options[:add_on] ||= true
end

add_default_config(my_options) # => Object has id: 60

puts my_options # => { :widget => true, :add_on => true }
```

It just works because both the local variable `my_options` in the main scope and the local variable `options` in the method scope are referencing the same object. Calling a method by referencing it via any of those two variables will change the same object.

What happens if I try to assign a new object to the local variable `options` ?

```ruby
my_options = { widget: true }
puts my_options.object_id # => 60

def add_default_config(options)
  puts "Object has id: #{options.object_id}"
  options = {}
  puts "Object now has id: #{options.object_id}"
  options[:add_on] = true
end

add_default_config(my_options)
# will print
# => Object has id: 60
# => Object now has id: 80

puts my_options # => { :widget => true }
```

If we try to assign a new object via hash literal `{}` inside the method `add_default_config` that will create a new object and assign that object to the variable called `object` that exists inside the method scope. This means that the local variable `options` in the method scope is now referencing a new object with an id `80` while the local variable `my_options` in the main scope is still referencing the object with id `60`.

```plaintext
+--------------------+------------+-----------+
| lexical scope      | label      | object id |
|--------------------|------------|-----------|
| main               | my_options | 60        |
| add_default_config | options    | 80        |
+--------------------+------------+-----------+
```

## Is Ruby pass-by-value or pass-by-reference?

I think the best answer is given by [Yukihiro Matsumoto](https://en.wikipedia.org/wiki/Yukihiro_Matsumoto) (Matz) and David Flanagan in the book called ["Programming Ruby"](https://www.oreilly.com/library/view/the-ruby-programming/9780596516178/) (O'Reilly, 2008):

!["When we work with objects in Ruby, we are really working with object references. It is not the object itself we manipulate but a reference to it. When we assign a value to a variable, we are not copying an object “into” that variable; we are merely storing a reference to an object into that variable"](https://cdn.hashnode.com/res/hashnode/image/upload/v1688527578475/b3a154e3-2ebc-4c84-8d22-248947ccc15e.png align="center")

!["When you pass an object to a method in Ruby, it is an object reference that is passed to the method. It is not the object itself, and it is not a reference to the reference to the object. Another way to say this is that method arguments are passed by value rather than by reference, but that the values passed are object references.  Because object references are passed to methods, methods can use those references to modify the underlying object. These modifications are then visible when the method returns"](https://cdn.hashnode.com/res/hashnode/image/upload/v1688530030380/8b38a69c-4bd5-4417-bd4c-ce469109b838.png align="center")

And I would add to this [the following quote](https://launchschool.medium.com/object-passing-in-ruby-pass-by-reference-or-pass-by-value-6886e8cdc34a) that describes how to think about the reference itself and why `options = {}` inside the method will not change the reference outside the method:

[!["While we can change which object is bound to a variable inside of a method, we can’t change the binding of the original arguments. We can change the objects if the objects are mutable, but the references themselves are immutable as far as the method is concerned"](https://cdn.hashnode.com/res/hashnode/image/upload/v1688530155046/4a082b73-7d83-4be1-a8a7-d9783442375c.png align="center")](https://launchschool.medium.com/object-passing-in-ruby-pass-by-reference-or-pass-by-value-6886e8cdc34a)

Coming back to the definition of pass-by-value and pass-by-reference, I like to think about it in the following way:

* Ruby is `pass by value` because when passing an argument to a method it will pass a copy of the reference to the object and not the reference itself
    
* Thus Ruby is `pass by value where the value is a reference to an object`
    

And implications of this are:

* Because the reference is pointing to an object, changing the object via the reference inside the method will be visible outside the method
    
* Because what is passed is a copy of the reference and not the reference/pointer itself, we cannot change the reference. This is why doing `parameter = Object.new` inside a method will not change the variable outside the method to point to a new object.
    

Or simply put Ruby is `pass-reference-by-value` as defined by Robert Heaton in his very nice article [Is Ruby pass-by-reference or pass-by-value?](https://robertheaton.com/2014/07/22/is-ruby-pass-by-reference-or-pass-by-value/)

In conclusion, Ruby uses a pass-reference-by-value approach when passing objects to methods. This means that while methods can modify the objects they receive, they cannot change the original references. Understanding this concept is crucial for effectively working with objects in Ruby and avoiding unexpected behaviour.

## More to read

If you want to read more about this and see other examples here are some good resources:

* [Is Ruby pass by reference or by value? - StackOverflow](https://stackoverflow.com/questions/1872110/is-ruby-pass-by-reference-or-by-value) ([Internet Archive](https://web.archive.org/web/20220522104444/https://stackoverflow.com/questions/1872110/is-ruby-pass-by-reference-or-by-value?noredirect=1&lq=1) version)
    
* [Is Ruby pass-by-reference or pass-by-value?](https://mixandgo.com/learn/ruby/pass-by-reference-or-value) ([Internet Archive](https://web.archive.org/web/20230323231901/https://mixandgo.com/learn/ruby/pass-by-reference-or-value) version)
    
* [Is Ruby Pass-by-Value Or Pass-by-Reference?](https://www.infoq.com/articles/ruby-parameter-passing/) ([Internet Archive](https://web.archive.org/web/20230616203433/https://www.infoq.com/articles/ruby-parameter-passing/) version)
    
* [Object Passing in Ruby — Pass by Reference or Pass by Value](https://launchschool.medium.com/object-passing-in-ruby-pass-by-reference-or-pass-by-value-6886e8cdc34a) ( [archive.is](https://archive.is/tQDHg) version)
    
* [Is Ruby pass-by-reference or pass-by-value?](https://robertheaton.com/2014/07/22/is-ruby-pass-by-reference-or-pass-by-value/) ([Internet Archive](https://web.archive.org/web/20230129220622/https://robertheaton.com/2014/07/22/is-ruby-pass-by-reference-or-pass-by-value/) version)
    
* [Ruby: pass by value or pass by reference](http://rubyblog.pro/2017/09/pass-by-value-or-pass-by-reference) ([Internet Archive](https://web.archive.org/web/20230325023052/http://rubyblog.pro/2017/09/pass-by-value-or-pass-by-reference) version)