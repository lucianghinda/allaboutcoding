---
title: "Two insights from using Sorbet"
seoTitle: "Sorbet usage: two key insights"
seoDescription: "How to fix the splat arguments error when using Sorbet and simplify code"
datePublished: Fri Feb 09 2024 04:06:52 GMT+0000 (Coordinated Universal Time)
cuid: clse4lsae000e09gy4u2yhy9g
slug: two-insights-from-using-sorbet
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1707451504809/6a9317ac-03eb-4ff3-9135-6c22ef7aa221.png
tags: ruby, ruby-on-rails, types, sorbet

---

I am working with Sorbet on a relatively big Ruby on Rails project, and here are two quick learnings:

1. It can help with simplifying code by removing unnecessary transformations.
    
2. The support that Ruby has for doing things in multiple ways is very helpful when hitting a case where Sorbet has a rough edge
    

Let's take them step by step:

## How to simplify methods by removing unnecessary transformations

Here is a piece of code that is a public interface of an object (this is not the actual code but an approximation of the actual code)

```ruby
# typed: strict

ACCEPTED_KEYS = T.let(%i[name username email].freeze, T::Array[Symbol])

sig { params(attributes: T::Hash[T.any(Symbol, String), Integer]).returns(T::Hash[Symbol, Integer]) }  
def accepted_attributes(attributes)  
  attributes
	  .symbolize_keys
	  .slice(*ACCEPTED_KEYS)
end
```

Let's first focus on the `.symbolize_keys` call and the definition of the `attributes` param that is `T::Hash[T.any(Symbol, String), Integer]` allowing the method to be called with a hash with keys, symbols, or strings.

Notice that because I defined the `attributes` in that way (to accept both types of keys), I called `symbolize_keys` to transform them into symbols so that I could `slice` by `ACCEPTED_KEYS`, which are symbols.

But there is another way to think about this:

*Because the method has type definitions, there is no need to support both String and Symbol as keys for the hash. The developer who plans to use this method can see exactly what kind of objects the keys should be.*

So, the method can be transformed into a simpler one where I remove the `symbolize_keys` :

```diff
- sig { params(attributes: T::Hash[T.any(Symbol, String), Integer]).returns(T::Hash[Symbol, Integer]) }  
+ sig { params(attributes: T::Hash[Symbol, Integer]).returns(T::Hash[Symbol, Integer])} 
def accepted_attributes(attributes)  
  attributes
-	  .symbolize_keys
	  .slice(*ACCEPTED_KEYS)
end
```

An insight here is that Sorbet can help remove code that is there to handle the cases where the input is in various formats. Using types will provide hints to the user of the method about what kind of arguments are expected, so there is no need to handle multiple types.

## Fixing the Sorbet splat arguments not well supported

Now, if you run the sorbet checker or play online on the sorbet playground, you will get an error that is something like this:

```ruby
editor.rb:10: Splats are only supported where the size of the array is known statically https://srb.help/7019
    10 |  attributes.slice(*ACCEPTED_KEYS)
```

And checking [the Sorbet docs](https://sorbet.org/docs/error-reference#7019) they say the following:

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707451254452/e4a13ebd-03e6-446e-8878-7edf62f1dcec.png align="center")](https://sorbet.org/docs/error-reference#7019)

One solution would be to use `T.unsafe` but I don't want to do that unless there is no other way around.

After trying a couple of things, I finally decided to refactor it to:

```ruby
sig { params(attributes: T::Hash[Symbol, Integer]).returns(T::Hash[Symbol, Integer]) }  
def accepted_attributes(attributes)  
  attributes.select { |key, _| ACCEPTED_KEYS.include?(key) }
end
```

And here comes the second insight:

*The amazing thing about Ruby is that I can refactor that method from using*`slice`*to using*`select`*and still remain explicit and read like an English sentence.*