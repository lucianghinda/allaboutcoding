---
title: "Overriding Methods in Ruby on Rails: A No-Code-Editing Approach"
seoTitle: "Overriding Methods in Ruby on Rails: A No-Code-Editing Approach"
seoDescription: "Discover how to override a class or instance method without editing its source file in Ruby on Rails."
datePublished: Wed Dec 04 2024 09:14:25 GMT+0000 (Coordinated Universal Time)
cuid: cm49o8zpl002b0amh76bu07o6
slug: overriding-methods-in-ruby-on-rails-a-no-code-editing-approach
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1733161881695/6d557e65-899e-4276-bd0f-9a3da8edb60f.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1733303220547/326a3172-1fae-44c8-9687-6c1c332dc7bc.png
tags: programming-blogs, ruby, ruby-on-rails

---

### The context

Recently, I installed the [Writebook](https://once.com/writebook) a Ruby on Rails app from [37signals](https://37signals.com) because I wanted to publish some long essays or long-form articles about testing for developers.

The installation worked like a charm, including replacing Rescue with Solid Queue and adding Solid Cable. I installed it manually, not via the `once` tool as I already have other projects on that specific server and did not had enough time to think how deploying it via the provided tool will affect the other projects. This means I would have to update it when an update came around manually.

It is currently published at [https://booklet.goodenoughtesting.com](https://booklet.goodenoughtesting.com), and I suggest checking it out if you are interested in improving your testing skills as a developer.

During the installation, I was pleasantly surprised to find that the Writebook app automatically creates a new book, which is the manual of the Writebook itself, at the first run. This is a great user experience, having the manual as the first book.

But when I wanted to create my book, I noticed that the URL was:

`https://booklet.goodenoughtesting.com/2/my-own-book`

Notice the `2` in that URL? I did not want that.

I could have deleted everything, reset the auto-increment, disabled the foreign key check for SQLite, and updated my book's ID. I have used these techniques on various occasions in the past, and while effective, they are solutions that I would only use if required. Messing with DB increment and foreign keys should be a solution only when everything else fails.

### The easiest solution - to comment out the code

The most straightforward solution was to open the log file and see what was executed. I found that the FirstRunsController executes `FirstRun#create!`, which then creates an account and calls `DemoContent.create_manual`.

Side note, and maybe to the surprise of some people: the `first_run.rb` file can be found in `app/models/first_run.rb`, but it is not an Active Record model.

[Rails Guides](https://guides.rubyonrails.org/getting_started.html#mvc-and-you-generating-a-model) agrees with this, but this is a side note.

![A screenshot from Rails Guides saying that a model is a Ruby class](https://cdn.hashnode.com/res/hashnode/image/upload/v1733148356965/77d337c0-6f44-4852-88d8-421dd9bd5e80.png align="center")

Let’s go back to WriteBook and try not to have the first book created automatically.

I could have just added comments for that call to `DemoContent`, but then I could have stopped there and learned nothing but to resolve the problem without spending too much time and refresh some things about Ruby and Rails 😀

I decided not to edit the Writebook code too much, as I wanted to be able to update it easily. I remember hacking the core WordPress code in 2006/2007 and having to re-apply all my changes every time there was an upgrade. I want to avoid that experience here.

### The beauty of Ruby as a dynamic programming language

Thinking about Ruby as a dynamic programming language and Rails as a long-standing web framework and knowing both of them allows you to do almost any crazy thing you can think of in terms of customization.

So, the simplest solution for me would be to add a file in the `config/initializers` that will override `DemoContent#create_manual`. This way, all the Writebook code remains untouched, except this file, making the upgrade process simple.

## Hooking into Rails initialization

After looking into [all the initialization events for Rails 8](https://guides.rubyonrails.org/configuring.html#initialization-events) and talking with my friend [Adrian](https://adrianthedev.com) I decided to use `to_prepare`, which is described as:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733148842974/938272cc-2e38-4299-ab35-c995d45a5ff8.png align="center")

and so I wrote the simplest code in an intializer:

```ruby
# config/initializers/personal_overrides.rb

Rails.application.config.to_prepare do
  class DemoContent
    def self.create_manual(user)
      Rails.logger.info "Overriding create_manual"
    end
  end
end
```

This just opens the DemoContent class method (after it was loaded) and will add a logger info without doing anything else.

It worked and after I did a `rails db:reset` and opened the web app, there I had the beautify of nothing: no book created. Which is wat I was trying to solve so now my book has the `id` = `1`.

By the way a great talk about the Rails boot process is this one from [Xavier Noria](https://hashref.com) at Rails World: [https://www.youtube.com/watch?v=Kx0ihLCTEgE](https://www.youtube.com/watch?v=Kx0ihLCTEgE)

## Other ways to override a class method

There I asked myself, what are other waysto override a class method without editing the file where the class is defined and I got a few ideas. Probably there are much more then the ones I tried here.

### Three ways to use `prepend`

Among the first things, I remembered that I don’t use `prepend` too much.

1. Defining a simple module and then using prepend on the `singleton_class` which basically means that we want the methods from `DemoContentOverride` to be called first when they are found and we use `singleton_class` to inject them into the class methods.
    

```ruby
Rails.application.config.to_prepare do
  module DemoContentOverride
    def create_manual(user)
      Rails.logger.info "Overriding create_manual"
    end
  end

  DemoContent.singleton_class.prepend(DemoContentOverride)
end
```

2. We can of course open the `DemoContent` class, define a method inside it and prepand that module from within the class
    

```ruby
Rails.application.config.to_prepare do
  DemoContent.singleton_class.class_eval do
    module DisableCreateBook
      def create_manual(user)
        nil
      end
    end

    prepend DisableCreateBook
  end
end
```

3. Doing the same as in step 2 but with an anonomious module:
    

```ruby
Rails.application.config.to_prepare do
  DemoContent.singleton_class.prepend(Module.new do
    def create_manual(user)
      Rails.logger.info "Overriding create_manual"
    end
  end)
end
```

### Using `undef_method` or `remove_method`

Another solution is just to remove the method and then create another one.

Here `undef_method` and `remove_method` can be used (for our need they are doing similarish things, but there is difference there in case there is an inheritance chain):

```ruby
Rails.application.config.to_prepare do
  DemoContent.singleton_class.undef_method(:create_manual)
  DemoContent.singleton_class.define_method(
    :create_manual,
    lambda { |user| Rails.logger.info "Overriding create_manual" }
  )
end

# or

Rails.application.config.to_prepare do
  DemoContent.singleton_class.undef_method(:create_manual)
  DemoContent.singleton_class.define_method(
    :create_manual,
    ->(user) { Rails.logger.info "Overriding create_manual" }
  )
end
```

### Using the singleton class notation `class « <Class>`

```ruby
Rails.application.config.to_prepare do
  class << DemoContent
    def create_manual(user)
      Rails.logger.info "Overriding create_manual"
    end
  end
end
```

The `class « DemoContent` from here is similar with doing:

```ruby
class DemoContent
  class << self
    def create_manual(user)
      Rails.logger.info "Overriding create_manual"
    end
  end
end
```

This is my exploration so far that at least for me, shows the power of Ruby language and the beauty of having access to the code of the web app that I use.

## What about an instance method

A similarish approach works for instance methods too. But you don’t need to use the singleton/Eigenclass for that. You can just `prepend` or `undef_method` on the instance methods and there is no need to work with the class singleton/eigenclass.

Say the DemoContent would have had the following structure:

```ruby

class DemoContent
  def initialize(user)
    @user = user
  end

  def create_manual
    # ...
  end
end
```

So the call for it would have been `DemoContent.new(user).create_manual`.

In this case I would need the following in my initializer if I wanted to just open the class after it was loaded and change the instance method:

```ruby
Rails.application.config.to_prepare do
  class DemoContent
    def create_manual
      Rails.logger.info "Overriding create_manual"
    end
  end
end
```

or write this if I wanted to use the `prepend`:

```ruby
Rails.application.config.to_prepare do
  module TestObjectOverride
    def original_method
      Rails.logger.info "Overriding create_manual"
    end
  end

  TestObject.prepend(TestObjectOverride)
end
```

or this if you want to use `undef_method` and redefine it again with `define_method`:

```ruby
Rails.application.config.to_prepare do
  DemoContent.undef_method(:create_manual)

  DemoContent.define_method(
    :create_manual,
    ->(user) { Rails.logger.info "Overriding create_manual" }
  )
end
```

---

👉 If you want to learn test case design and write fewer tests while covering more features, I invite you [to one of my 3-hour live online workshop](https://goodenoughtesting.com).

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733303507860/04f6a3e1-4107-4529-a9da-0e79e51f4e49.png align="center")](https://goodenoughtesting.com)