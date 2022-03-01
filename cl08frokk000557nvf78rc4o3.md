## How to compare two arrays of hashes by value in Ruby

# Context

Let's say that you have two array of hashes:

```ruby
a = [ 
  {
    initial: "a", 
    email: "a@example.com"
  }, 
  {
    email: "b@example.com", 
    initial: "b"
  }, 
  {
    initial: "c", 
    amount: 3, 
    email: "c@example.com"
  }
]
```

and

```ruby
b = [
  {
    email: "c@example.com",
    amount: 3,
    initial: "c"
  },
  {
    email: "b@example.com",
    initial: "b"
  },
  {
    email: "a@example.com",
    initial: "a"
  }
]
```
Maybe they are from an import you are doing or maybe you received them via a call from a JSON structure or you just called `to_h` on an object. 

# Problem

You want to verify if these two arrays contain the same values inside their hash elements and execute `a == b`.

You will see quickly the response as being `false`. 

Here is the [reason from Ruby 3.1 docs](https://ruby-doc.org/core-3.1.0/Array.html#method-i-3D-3D) (but it works the same also in 3.0, 2.7, 2.6 ...):

![Array#== exaplanation](https://cdn.hashnode.com/res/hashnode/image/upload/v1646142961609/5noniGUIS.png)

But you look at the two arrays and see what their values are the same, just that they are not in the same place in both of them.

# Solutions

There are probably multiple solutions to this. 
I will present two 

## (a - b) + (b-a) == []

This solution uses the difference and union to check that the difference between the two arrays does not contain any elements. 

The difference between the two arrays will return all elements in the first array that are not present in the second array. See [`Array#-`](https://ruby-doc.org/core-3.1.0/Array.html#method-i-2D) 

So this solution simply makes sure that `a - b` is empty and `b - a` is also empty.

You can rewrite this in multiple ways: 

```ruby
((a-b) + (b-a)) == []

(a-b).empty? && (b-a).empty?

((a-b) + (b-a)).empty?
```

## Using Set class

This solution takes into consideration two properties of `Set` in Ruby:

1. `Set.new` with an enumerable as an argument will create a Set containing all items from inside the enumerable object.
2. Sets by definition does not take into account the order in which the elements were added to the set. 

Based on this you might say that the solution is to simply execute: 

```ruby
Set.new(a) == Set.new(b)
```
But this has a hidden bug due to another property of a set: one element is included only once in a set, no matter how many times we try to add it. 

For example, this scenario will fail: 

```ruby
m = [1,2,2]
n = [2,1]

puts Set.new(m) == Set.new(n) #=> returns true !!!
```

So the proper solution is the following:

```ruby
m.length == n.length && Set.new(m) == Set.new(n)
```



