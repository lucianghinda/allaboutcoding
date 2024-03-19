---
title: "History of the endless method syntax"
seoTitle: "Endless Method Syntax History"
seoDescription: "Explore the evolution of Ruby's endless method syntax: its origins, proposals, and the innovative solution for concise, single-expression methods"
datePublished: Tue Mar 19 2024 07:37:03 GMT+0000 (Coordinated Universal Time)
cuid: clty2ab48000b08l30ndv05iq
slug: history-of-the-endless-method-syntax
canonical: https://learn.shortruby.com/blog/history-of-endless-method
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1710833696841/fb9d58ca-1613-47df-971b-df072e7ad09e.png
tags: programming-blogs, ruby

---

When I learn about a new language feature, I like to read and discuss the proposal. How and why it was accepted. What was the requester trying to accomplish, and what problem did they try to solve?

Here, I will review how the endless method was introduced in the Ruby language.

## What is the definition of the endless method?

In a few words, it is an alternative syntax for defining a method that consists of a single expression.

Here is the definition from [Ruby documentation](https://docs.ruby-lang.org/en/master/syntax/methods_rdoc.html):

![Definition of endless method in Ruby](https://cdn.hashnode.com/res/hashnode/image/upload/v1710833147101/c87c957c-88ea-4ffe-b0a8-bd698cd12157.png align="center")

Here are two simple examples:

```ruby
def exists? = User.where(organisation: organisation).exists?
```

or with parameters:

```ruby
def format_date(date) = date.strftime(DEFAULT_DATE_FORMAT)
```

## History of trying to remove one or more `end`s

### 2011 - the first proposal is made mainly as a joke, but it tries to solve a real problem with idiomatic code

In 2011, Yasushi Ando [proposed](https://bugs.ruby-lang.org/issues/5054) (as a joke) an `en(n+)d` to solve the following issue: having too many ends:

```ruby
# Source: https://bugs.ruby-lang.org/issues/5054

# Replace this code:

module MyModule
  class MyClass
    def my_method
      10.times do
        if rand < 0.5
          p :small
        end
      end
    end
  end
end

# With:
module MyModule
  class MyClass
    def my_method
      10.times do
        if rand < 0.5
          p :small
        ennnnnd
```

While the proposal was a joke, the idea is that sometimes the number of `end`s we have to write is high (sometimes it takes more chars than the actual business logic).

Another [proposal](https://bugs.ruby-lang.org/issues/5065) (this time a serious one) was made by Lazaridis Ilias later the same year, 2011. This time the proposal was to allow `}` instead of `end`:

```ruby
# Source: https://bugs.ruby-lang.org/issues/5065

module MyModule
  class MyClass
    def my_method
      10.times {   # "10.times do" would work, too
        if rand < 0.5
          p :small
        }
      }
    }
  }
}
```

This one was rejected. Among other proposed ideas was to have an `endall` as a keyword that will add all necessary ends if I might say so:

```ruby
# Source: https://bugs.ruby-lang.org/issues/5065

module MyModule
  class MyClass
    def my_method
      10.times do
        if rand < 0.5
          p :small
        endall
```

### 2016 - another proposal was made, this time a serious one

Then, in 2016, Nobuyoshi Nakada [proposed](https://bugs.ruby-lang.org/issues/12241) another idea, building upon the previous one: to introduce the `end!` keyword like a `super end` concept that will end all blocks not ended until its level:

```ruby
# Source: https://bugs.ruby-lang.org/issues/12241

module MyModule
  class MyClass
    def my_method
      10.times do
        if rand < 0.5
          p :small
!end
```

There were few replies to it, so there was no final decision.

### 2020 - a new proposal with more arguments and a simple approach is suggested

The [feature request](https://bugs.ruby-lang.org/issues/16746) for the current format was submitted in 2020 by Yusuke Endoh on the Ruby issue tracker:

![Endless method definition - proposal from Yusuke in Ruby tracker](https://cdn.hashnode.com/res/hashnode/image/upload/v1710833182248/667626a6-62c8-4b25-89e8-e97825a8f547.png align="center")

The initial proposal wanted to use `:` instead of `=` when defining the method body, but Matz [proposed](https://bugs.ruby-lang.org/issues/16746#note-7) using `=`. Still, the go-ahead for running an experiment like this was given at the end of 2020.

Later in that thread, Yusuke [presented](https://bugs.ruby-lang.org/issues/16746#note-8) some arguments:

> Surprisingly, according to the following rough estimate of ruby/ruby code base, this kind of simple method definitions account for 24% of the entire method definitions.

Victor Shepelev [added](https://bugs.ruby-lang.org/issues/16746#note-25) some excellent arguments in favor of this proposal:

> The most precious Ruby's quality for me is "expressiveness at the level of the single 'phrase' (expression)", and it is not "code golf"-expressiveness, but rather structuring language's consistency around the idea of building phrases with all related senses packed, while staying lucid about the intention

> I believe that the difference of "how you write it when there is one statement" vs "...more than one statement" is intentional, and fruitful: "just add one more line to 10-line method" typically small addition, but "just add one more line to 1-line method" frequently makes one think: may be that's not what you really need to do, maybe data flow becomes unclear

### 2021 - a new proposal is made for a super-end keyword

While the status of the Yusuke proposal kept having multiple arguments, and Matz already proposed a syntax there that he would like but said might be conflicting, a new proposal was submitted trying to solve the same issue but focusing on nested ends.

This [proposal](https://bugs.ruby-lang.org/issues/17786) added by Jabari Zakiya is about the same idea of how to reduce the number of `end` keywords, and in this case, the proposal was to use `ends`:

```ruby
# Source: https://bugs.ruby-lang.org/issues/17786

def render(scene, image, screenWidth, screenHeight)
  screenHeight.times do |y|
    screenWidth.times do |x|
      color = self.traceRay(....)
      r, g, b = Color.toDrawingColor(color)
      image.set(x, y, StumpyCore::RGBA.from_rgb(r, g, b))
    end
  end
end

# To be replaced by

def render(scene, image, screenWidth, screenHeight)
  screenHeight.times do |y|
    screenWidth.times do |x|
      color = self.traceRay(....)
      r, g, b = Color.toDrawingColor(color)
      image.set(x, y, StumpyCore::RGBA.from_rgb(r, g, b))
ends
```

This, too, not approved yet. Notice this is a bit of a tangent to the endless method, but it is worth considering that the idea that the number of `end` keywords written should be limited is present.

### Ruby 3.0 - release on 25 December 2020 included endless method definition

Ruby version 3.0 included the endless method definition or shorthand method syntax with the following format:

```ruby
def <name>(<params>) = <body>
# or
def <name>(<params>) = <an expression
	that can be on
	multiple lines>
# or
def <name> = <body>
# or
def <name> = <an expression
	that can be on
	multiple lines>
```

Of course, having arguments is optional, but if you decide to specify them, then parentheses are mandatory.

## More about the endless method

One of the best resources to read about this in a format that does an excellent assessment of this feature is the article written by Victor Shepelev called [“Useless Ruby sugar”: Endless (one-line) methods](https://zverok.space/blog/2023-12-01-syntax-sugar5-endless-methods.html)

## Curious to learn more about this?

If you want to learn more about this feature, its gotchas, when and how to use it, and real code examples, I am creating a course called [“Modern Ruby Syntax”](https://learn.shortruby.com/courses/modern-ruby), where I explore the endless method along with other Ruby features.

---

**Enjoyed this article?**

👐 Subscribe to my Ruby and Ruby on rails courses over email at [**learn.shortruby.com**](http://learn.shortruby.com/)**\- effortless learning about Ruby anytime, anywhere**

👉 Join my [**Short Ruby News**](https://shortruby.com/) **newsletter** for weekly Ruby updates from the community and visit [**rubyandrails.info**](http://rubyandrails.info/)**, a directory with learning content about Ruby.**

🤝 Let's connect on [**Ruby.social**](http://ruby.social/) or [**Linkedin**](https://linkedin.com/in/lucianghinda) or [**Twitter**](https://x.com/lucianghinda) where I post mainly about Ruby and Rails.

🎥 Follow me on [**my YouTube channel**](https://www.youtube.com/@shortruby) for short videos about Ruby