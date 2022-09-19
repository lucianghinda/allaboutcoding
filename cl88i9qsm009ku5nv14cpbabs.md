## What is a class in Ruby?

I will try to explain what a class is in some other way, somehow quoting or paraphrasing in some parts what [@steveklabnik](https://twitter.com/steveklabnik) said a very long time ago in a presentation about OOP in Ruby.

Plato had a theory called "A Theory of Forms and Ideas" which is something like this:

Think about a perfect triangle as described by a mathematical formula. This would be a description of a form or idea of a triangle.

Plato says this triangle exists in an abstract and perfect state, but it is somehow independent of our minds. These Ideas or Forms live in their own realm, which is not our universe. 

We might try to draw a triangle, but this will be an (imperfect) representation of the triangle in our universe.

Still, almost no matter how badly we can draw a triangle, anyone can recognize it as a triangle if it has the general shape:


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663575805457/r-n-hqdUB.png align="left")

Thus we draw a triangle shape, and even if imperfect, we can work with it as a triangle that works out. We can compose other objects, we can calculate various parts of them, can split them into smaller objects, and much more. 

A class is like a Form or Idea of an object. 
We think about what object we will like to create, and then in the realm of Forms and Ideas we say a definition of it that usually contains data this ideal object might have and how it will behave. 

We have a notation for this in Ruby: 

```ruby
class Triangle
end
```

**Thus, a class represents the Idea of how we want an object to be.**

Then we instantiate an object that looks like the idea we wanted to describe. 
But it is not precisely that idea. 

We draw that idea in our universe where now we can move it around, add data to it & so on. 

```ruby
triangle = Triangle.new
```

The `triangle` will behave like the `Triangle`(the idea) but will not be the Idea itself.