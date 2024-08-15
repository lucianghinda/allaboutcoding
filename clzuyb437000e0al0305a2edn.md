---
title: "Proposal for making private keyword work on constants too"
seoTitle: "Proposal to change private Keyword to work on Constants in Ruby"
seoDescription: "Proposal to reopen discussion on making the `private` keyword work on constants in Ruby for better code simplicity and consistency"
datePublished: Thu Aug 15 2024 07:20:38 GMT+0000 (Coordinated Universal Time)
cuid: clzuyb437000e0al0305a2edn
slug: proposal-for-making-private-keyword-work-on-constants-too
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1723702639816/b19de5e7-1af0-4e89-9713-e9e0483d123b.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1723702683205/c7b27b14-7eb3-41b5-86fc-2c86d8d567cf.png
tags: code, programming-blogs, ruby, ruby-on-rails

---

There was a proposal on the Ruby bug tracker a while back about making the `private` keyword work also on constants:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723695827199/a5cec95d-bfc3-4a39-a4e0-cd45f4933b66.png align="center")

See [https://bugs.ruby-lang.org/issues/17171](https://bugs.ruby-lang.org/issues/17171)

That proposal is now closed with concerns about backward compatibility.

We should reopen this discussion and try to find ways around backward compatibility. Let me explain why I think so:

## A possible code in Ruby

This might be a possible code design found in a Ruby project:

```ruby
class MyObject 
  DEFAULT_VALUE = 'active'
  private_constant :DEFAULT_VALUE
  
  def a_public_method
    # doing something important here
  end
end
```

When reading this file, we first read DEFAULT\_VALUE then we find out it is a private constant and then we go to read the actual public interface. I would assume that usually the information architecture that we would want when reading this file would be something like this:

1. Public methods
    
2. Public constants
    
3. Private methods
    
4. Private constants
    

Let's take two possible cases for reading this file:

* Trying to understand what this object does: in this case I want to see first the public method(s) and then go down inside private methods if I want to expand my knowledge and understand some fine grained details.
    
* Trying to understand what is the content of a public constant called from some other place: In this casde I want to see first the public constant
    

Anyhow no matter the objective I almost never want to read first the private methods or private constants.

Going to private constants: I don't think there is a good case to be made for putting a private constant in the public space of the object. The constant is private because it should be used only inside the current object in its either public or private methods.

So maybe the following code design is a better fit from the point of information architecture:

```ruby
class MyObject 
  def a_public_method
    # doing something important here
  end
  
  private 
  
    DEFAULT_VALUE = 'active'
    private_constant :DEFAULT_VALUE
end
```

Of course in this case while the intention of the information is communicated pretty clear and the code is organised in public interface to the top and private interface to the bottom, there is a duplicate information there:

`private` and `private_constant`

But moreso it is a confusing thing that when first learning Ruby you just have to learn it: the `private` keyword does not affect constants so you have to use `private_constant`.

I think a better (in the sense of simplicity and expectations) code design would be to write:

```ruby
class MyObject 
  def a_public_method
    # doing something important here
  end
  
  private 
    DEFAULT_VALUE = 'active'
end
```

And make Ruby `private` method to affect also constants. So `MyObject::DEFAULT_VALUE` should throw `NameError` .

## Arguments in favor of this feature

1. **Principle of least surprise**
    

The main argument for making the `private` keyword work for me would be based on the principle of least surprise: seeing a code like the one above will make everybody think that `DEFAULT_VALUE` is a private constant. So it should work that way.

2. **Removing boilerplate**
    

The second argument is that it will remove a boilerplate code: the `private_constant` method should be used in a small number of situations. It seems to me that it should remain in the language but for specific usage.

3. **Removes the need to know a trick (maybe still principle of least surprise)**
    

When encountering the keyword `private` the expectation is that what follows is something private. And writing a constant there is now making it public, disrupting the flow of thoughts aobut the visibility of what is in that section of the code.

## Compatibility concerns

I know that the [previous feature request](https://bugs.ruby-lang.org/issues/17171) was closed with a concern for breaking compatibility by Matz. Which is a valid concern and I appreciate so much the case he puts into making our life easier when upgrading.

But I do wonder if in this case the worrying is real.

What is the main case of implementing breaking changes? When someone added the constant in the `private` section or called the `private` method for it and then used it as being public.

Something like this:

```ruby
class MyObject 
  def a_public_method
    # doing something important here
  end
  
  private 
    DEFAULT_VALUE = 'active'
end

class AnotherObject 
  def some_method
    MyObject::DEFAULT_VALUE
  end
end
```

I do wonder two things:

1. How common is this way of writing code: putting the declaration of a private constant in the private section of an object?
    
2. And if this exists how common it is for the constant defined like that to be accessed outside that object?
    

My main assertion is that if someone writes a constant in the `private` area of an object, their intention (even if it is not actually enforced during execution) is that they want to say or show that constant is private.

Calling that constant from other objects is breaking this contract. It is for me the same as calling `send` or `__send__` on a private method. It can be done but that breaks the contract for that object. But if want to achieve the same for cosntants we already have `const_get` that will get any constant from an object.

Since we have this repo now [Gem CodeSearch](https://github.com/akr/gem-codesearch) I will try to see at least for gems if I can find a way to search for this specific case (using a constant defined as `private` - but not with `private_constant` into another object).

## New functionality

I think making the `private` work on constants will open the case for making a class defined inside another class private.

Take this code for example:

```ruby
class MyObject
  class AnotherObject
    def hello
      puts "I am here"
    end
  end

  def a_public_method
    AnotherObject.new.hello
  end
end

MyObject.new.a_public_method
```

I think `AnotherObject` remains public here unintentionally.

Most of the time this code should be:

```ruby
class MyObject
  class AnotherObject
    def hello
      puts "I am here"
    end
  end

  private_constant :AnotherObject

  def a_public_method
    AnotherObject.new.hello
  end
end

MyObject.new.a_public_method
# => 'I am here'

MyObject::AnotherObject.new.hello
# => private constant MyObject::AnotherObject referenced (NameError)
```

But people usually forget to add the `private_constant` here. And even so this goes to the first point where when reading this file, I first read an object that is private.

I think making the `private` keyword affect constants, will open the case to write something like this:

```ruby
class MyObject
  def a_public_method
    AnotherObject.new.hello
  end

  private
    class AnotherObject
      def hello
        puts "I am here"
      end
    end
end

MyObject.new.a_public_method
# => 'I am here'

MyObject::AnotherObject.new.hello
# => private constant MyObject::AnotherObject referenced (NameError)
```

So we could have private objects that can be defined with `private` method and be accessible only to the object where they are defined.

## What's next

I am talking mostly from the perspective of someone that uses the language, so I for sure don't know all the possible problems this change will create. But I think we should have another conversation about this. And see if we can make it work.

---

**Enjoyed this article?**

👉 Join my [**Short Ruby Newsletter**](https://newsletter.shortruby.com/) for weekly Ruby updates from the community and visit [**rubyandrails.info**](http://rubyandrails.info/)**, a directory with learning content about Ruby.**

👐 Subscribe to my Ruby and Ruby on Rails courses over email at [**learn.shortruby.com**](http://learn.shortruby.com/)\- effortless learning about Ruby anytime, anywhere

🤝 Let's connect on [**Ruby.social**](http://ruby.social/) **or** [**Linkedin**](https://linkedin.com/in/lucianghinda) **or** [**Twitter**](https://x.com/lucianghinda) where I post mainly about Ruby and Ruby on Rails

🎥 Follow me on [**my YouTube channel**](https://www.youtube.com/@shortruby) for short videos about Ruby