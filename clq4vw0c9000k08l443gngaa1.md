---
title: "Exploring `it` default block param warning in Ruby 3.3"
seoTitle: "Ruby 3.3 Examining `it` Default Block Param"
seoDescription: "Discover Ruby 3.3's `it` default block param warning, its impact on RSpec, and how to fix potential conflicts. Learn more in my video or article."
datePublished: Thu Dec 14 2023 07:33:32 GMT+0000 (Coordinated Universal Time)
cuid: clq4vw0c9000k08l443gngaa1
slug: exploring-it-default-block-param-warning-in-ruby-33
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1702538831850/b7bda507-7410-43cb-a064-539f8c738153.png
tags: programming-blogs, ruby, ruby-on-rails, coding

---

In the [last Ruby Developer Meeting](https://github.com/ruby/dev-meeting-log/blob/master/2023/DevMeeting-2023-11-30.md#feature-18980-re-reconsider-numbered-parameters-it-as-a-default-block-parameter-k0kubun) it was accepted that `it` will be added to Ruby 3.4.

In preparation for this, Ruby 3.3 will show a warning when `it` is used without a receiver, argument, or block.

I made a quick video about this if you want to watch it or if you prefer the written version continue reading on:

%[https://www.youtube.com/watch?v=DnMNn13k-rg] 

## Deprecation warning

For this, let's define some simple code where I will name a method `it` and then call it from a block:

```ruby
puts RUBY_VERSION

def it = "works"

def run(&block) = yield

puts run { it }
```

If you run this with Ruby 3.2.2 then you will get the following result:

```bash
3.2.2
works
```

But if you run this with Ruby 3.3.0.rc1 you will get the following result:

```bash
example_deprecation_warning.rb:7: warning: `it` calls without 
arguments will refer to the first block param in Ruby 3.4; 
use it() or self.it
3.3.0
works
```

## How to fix the warning

In case you have a method named `it` and still want to call it inside a block without a receiver, arguments or params you should either add paranthesis:

```diff
- puts run { it }
+ puts run { it() }
```

or add the receiver:

```diff
- puts run { it }
+ puts run { self.it }
```

## When using `it` as local variable name

There will be no problem when you defined `it` as local variable name.

Here is an example where I created:

```ruby
puts RUBY_VERSION

def log(&block) = puts yield

def run
  it = 2

  log { it }
end

run
```

If you run this it on Ruby 3.3.0.rc1 will not display an error. The output will be:

```bash
3.3.0
2
```

Now if you will extract the local variable to a method with the same name:

```diff
puts RUBY_VERSION

def log(&block) = puts yield
+
+ def it = 2
+
def run
-  it = 2

  log { it }
end

run
```

It will show a warning:

```bash
it_as_local_variable.rb:8: warning: `it` calls without arguments 
will refer to the first block param in Ruby 3.4; use it() or self.it
3.3.0
2
```

## RSpec

RSpec uses a lot `it`. But in most cases (I would argue majority of them) this will not be a problem. Because usually `it` is called with a string argument:

```ruby
require "rspec"

describe Galaxy do
  context "when created with a name and number of stars" do
    it "should hold the correct name and star count" do
      galaxy = Galaxy.new("Milky Way", 100_000_000_000)
      expect(galaxy.name).to eq("Milky Way")
      expect(galaxy.number_of_stars).to eq(100_000_000_000)
    end
  end
end
```

The code above will *NOT* display a warning.

But if you will try to write something like this:

```ruby
require "rspec"

describe Galaxy do
  subject { Galaxy.new("Milky Way", 100_000_000_000).stars }

  context "When there are no known stars" do
    it
  end
end
```

If you run this with Ruby 3.3.0.rc.1 you will get the warning:

```ruby
explore-it/rspec_example/spec/more_galaxy_spec.rb:8: warning: `it` 
calls without arguments will refer to the first block param in 
Ruby 3.4; use it() or self.it
*

Pending: (Failures listed here are expected and do not affect your suite's status)

  1) Galaxy When there are no known stars 
     # Not yet implemented
     # ./spec/more_galaxy_spec.rb:8


Finished in 0.0074 seconds (files took 0.06474 seconds to load)
1 example, 0 failures, 1 pending
```

You can fix this by using one of the other alias for `it` from RSpec:

* [`specify`](https://github.com/rspec/rspec-core/blob/f273314f575ab62092b2ad86addb6a3c93d6041f/lib/rspec/core/example_group.rb#L170)
    
* [`example`](https://github.com/rspec/rspec-core/blob/f273314f575ab62092b2ad86addb6a3c93d6041f/lib/rspec/core/example_group.rb#L158)
    

Here is [the complete list of methods](https://github.com/rspec/rspec-core/blob/f273314f575ab62092b2ad86addb6a3c93d6041f/lib/rspec/core/example_group.rb#L157C1-L157C1) defined in RSpec core:

```ruby
# Defines an example within a group.
define_example_method :example
# Defines an example within a group.
# This is the primary API to define a code example.
define_example_method :it
# Defines an example within a group.
# Useful for when your docstring does not read well off of `it`.
# @example
#  RSpec.describe MyClass do
#    specify "#do_something is deprecated" do
#      # ...
#    end
#  end
define_example_method :specify

# Shortcut to define an example with `:focus => true`.
# @see example
define_example_method :focus,    :focus => true
# Shortcut to define an example with `:focus => true`.
# @see example
define_example_method :fexample, :focus => true
# Shortcut to define an example with `:focus => true`.
# @see example
define_example_method :fit,      :focus => true
# Shortcut to define an example with `:focus => true`.
# @see example
define_example_method :fspecify, :focus => true
# Shortcut to define an example with `:skip => 'Temporarily skipped with xexample'`.
# @see example
define_example_method :xexample, :skip => 'Temporarily skipped with xexample'
# Shortcut to define an example with `:skip => 'Temporarily skipped with xit'`.
# @see example
define_example_method :xit,      :skip => 'Temporarily skipped with xit'
# Shortcut to define an example with `:skip => 'Temporarily skipped with xspecify'`.
# @see example
define_example_method :xspecify, :skip => 'Temporarily skipped with xspecify'
# Shortcut to define an example with `:skip => true`
# @see example
define_example_method :skip,     :skip => true
# Shortcut to define an example with `:pending => true`
# @see example
define_example_method :pending,  :pending => true
```

## Conclusion

I really like that `it` was [accepted](https://bugs.ruby-lang.org/issues/18980#note-47) as a default block parameter. Looking forward to Ruby 3.4 (next year) to write code using it.

The warning feature in Ruby 3.3 serves as a useful heads-up for potential conflicts with its usage, although these are expected to occur infrequently.

The transition should be smooth for most developers. RSpec tests should not require any changes, as it largely uses `it` with a string argument and is not likely to trigger the warning. In the instances where it does, alternatives like `specify` and `example` can be used to fix the warning.

---

Enjoyed this article?

👉 Join my [**Short Ruby News**](https://shortruby.com/) **newsletter** for weekly Ruby updates from the community and visit [**rubyandrails.info**](http://rubyandrails.info)**,** a directory with learning content about Ruby.

👐 Subscribe to my Ruby and Ruby on rails courses over email at [learn.shortruby.com](https://learn.shortruby.com) - effortless learning anytime, anywhere

🤝 Let's connect on [**Ruby.social**](https://ruby.social/@lucian) or [**Linkedin**](https://linkedin.com/in/lucianghinda) or [**Twitter**](https://x.com/lucianghinda) where I post mainly about Ruby and Rails.

🎥 Follow me on [**my YouTube channel**](https://www.youtube.com/@shortruby) for short videos about Ruby