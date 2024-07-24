---
title: "Hash value omission - an introduction and some examples"
seoTitle: "Understanding Hash Value Omission in Ruby"
seoDescription: "Learn about hash value omission in Ruby, their benefits, examples, and implementation guidelines for more concise code"
datePublished: Wed Jul 24 2024 07:16:14 GMT+0000 (Coordinated Universal Time)
cuid: clyzigpod001e09la4bbdhwze
slug: hash-value-omission-an-introduction-and-some-examples
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1721805296524/7ff5695e-5939-467c-8572-94af8cc5bd82.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1721805311357/f5a96c37-2f67-4404-8c9f-555488e5be65.png
tags: programming-blogs, ruby, rails, coding, programming-tips

---

The hash literal value omission was added to Ruby [version 3.1](https://www.ruby-lang.org/en/news/2021/12/25/ruby-3-1-0-released/) in 2021. This is the [feature proposal](https://bugs.ruby-lang.org/issues/14579) that describes the feature request and the one that was implemented.

## What does it mean?

Hash literal value omission (or hash value omission) means that `{ name: }` is a shortcut for `{ name: name }`

```ruby
url = 'https://shortruby.com'
source = 'email'

link = { url:, source: }
# is a shortcut for
link = { url: url, source: source }
```

This works also for method calls with keyword arguments:

```ruby
url = "https://ruby.social/@lucian"

result = UrlType.detect(url:)
# this is equivalent to writing
result = UrlType.detect(url: url)
```

And of course you can use it to pass values via keyword arguments to other methods when calling from inside current method:

```ruby
def get(path:)
  request(:get, path:)
end
```

It also searches through the methods inside the current object:

```ruby
class MetaTags
  def initialize(article:)
    @article = article
  end

  def to_meta_tags
    { title:, description:, keywords: }.compact_blank
  end

  private

    def title = "#{article.title} - #{CurrentSection.title}"
    def description = article.description
    def keywords = article.tags + [CurrentSection.keyword]
end
```

Notice there inside the method `to_meta_tags` there are no local variables named `title`, `description` or `keywords`, but there are private methods with that name. So when composing the hash, Ruby will look not only at local variables, but also try to find them among the methods of the current object.

What you see here is, in my opinion, the most powerful use of hash literal omission when combined with endless methods: a concise way to create a hash with some computed attributes while keeping the methods very short (one line).

## How it works?

The following code is an approximation (like a pseudocode) of how it works. I discovered this simple explanation by Akinori Musha in the [feature request](https://bugs.ruby-lang.org/issues/14579#note-32):

```ruby
# This is not the real code, but a possible explanation of 
# how it works if it would be implemented in Ruby
# Source: https://bugs.ruby-lang.org/issues/14579#note-32

value = if binding.local_variable_defined?(:user)
          binding.local_variable_get(:user)
        else 
          __send__(:user)
        end


{ user: value }
```

It also has a nice explanatory error message that describes in a way how it works.

Let's write a simple example inside a file called `error_message.rb`.

```ruby
# error_message.rb
my_hash = { name: }
```

When executed with `ruby error_message.rb`, it will display the following error message:

```bash
error_message.rb:1:in `<main>': undefined local variable or method `name' for main (NameError)

my_hash = { name: }
            ^^^^^
```

The error message is explicit:

* it says "undefined **local variable or method**", hinting on the fact that it tries to get the value for `name` either from a local variable or a method called `name`.
    

## Why use this feature?

Before going into my own explanation I think this [short reply](https://www.reddit.com/r/ruby/comments/s8j51q/comment/htgs353) from Victor Shepelev in Reddit makes a good compelling case for this feature:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721804241162/e6a884e8-5440-4ef9-86db-8cabaae70326.png align="center")

I will touch some of these points below, but I think Victor is doing here a great job explaining in a coincise way why this feature is not hard to understand or to use.

### Removes repetition

I think this kind of code can be easily found in multiple projects:

```ruby
user = User.find(params[:id]) # or some other way to find a user
authenticate(user: user, context: context)

# or 

def get(path, query_params: {})
  request(:get, path: path, query_params: query_params)
end

# or 

render partial: "user", locals: { user: user, organisation: organisation }
```

And with hash value omission it can be transformed into:

```ruby
user = User.find(params[:id]) # or some other way to find a user
authenticate(user:, context:)

# or 

def get(path, query_params: {})
  request(:get, path:, query_params:)
end

# or 

render partial: "user", locals: { user:, organisation:}
```

### A more general example

The following idiom is pretty common in a Rails application that uses keyword arguments:

```ruby
method(foo: foo, baz: baz, qux: qux)
```

It can easily go into a longer line:

```ruby
media_converter.create_job(role: role, settings: settings, queue: queue, project: project, input: input)
```

or this expression can be split into multiple lines:

```ruby
media_converter.create_job(
    role: role, 
    settings: settings, 
    queue: queue, 
    project: project, 
    input: input
)
```

There might be two issues with this type of code:

1. Repeating the same label multiple times
    
2. This makes the line longer and sometimes requires splitting it into multiple lines. A shorter version could help keep the code on a single line.
    

None of these indicate poor code quality. This feature is more about making things a bit better rather than fixing bad code.

In both cases, the code can be written in a more concise form:

```ruby
method(foo:, baz:, qux:)
media_converter.create_job(role:, settings:, queue:, project:, input:)
```

### Brings symmetry to pattern matching

When reading the feature request Matz [decides](https://bugs.ruby-lang.org/issues/14579#note-14) to accept the proposal with the following argument:

> After the RubyKaigi 2021 sessions, we have discussed this issue and I was finally persuaded.  
> Our mindset has been updated (mostly due to mandatory keyword arguments).  
> Accepted.
> 
> Matz.

It is not that clear from reading the notes there what was the *other* argument that convincend him along with keyword arguments.

Looking at [Dev Meeting Log](https://github.com/ruby/dev-meeting-log/blob/770f970c7dc05ceb02e65986d961485844dfd66b/2021/DevelopersMeeting20210916Japan.md?plain=1#L254C3-L254C66) the following was noted:

> matz: Yes, they should. I was convinced by the points knu made.

Thus I [asked](https://ruby.social/@lucian/112766409031867998) Akinori Musha via Ruby.social about what were the arguments and he [shared](https://ruby.social/@knu@mr.am/112774326818497440) the following:

![A post by Akinori MUSHA (@knu@mr.am) discusses the implementation of pattern-matching with value omission. It gives an example using  and explains that it is shorthand for .](https://cdn.hashnode.com/res/hashnode/image/upload/v1721718946896/0436b692-54fa-4925-a611-ed122173bea0.png align="center")

![A post by Akinori MUSHA describing how he convinced Matz to introduce value omission in hash literal syntax for symmetry. The tweet includes Ruby code defining  and  methods, and a method call showing  returns 42 when decomposed.](https://cdn.hashnode.com/res/hashnode/image/upload/v1721718977144/cf2aa606-1c54-4404-9f93-32c15a5db4b1.png align="center")

In this case hash value omission is seen as a composition, while pattern matching is using decomposition. I think it is an elegant way to think about this feature.

He also mentioned that the actual example he used in that discussion was one [shared](https://bugs.ruby-lang.org/issues/14579#note-13) also in the Ruby issue tracker where you can see the same idea of composition and decomposition in a single method:

```ruby
def get_user_profile(client)
  client.get_json("/current_user") => { id: }
  client.get_json("/profile", { id: }) => { nick:, bio: }

  return { id:, nick:, bio: }
end
```

### It is a known syntax

I know that the main argument against this syntax is that is is weird. And I read this weirdness more in the direction of not being familiar. But you can see this syntax in at least two places:

1. Keyword arguments
    
2. Pattern matching
    

Here is an example from **keyword arguments**:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721137669580/38a3cc8d-3b2d-4a99-a486-093fd95faef8.png align="center")

And here is an example from pattern matching (but in pattern matching this kind of syntax is common):

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721137709368/e185ec9f-5546-412d-b2c6-a6d6548b4b9f.png align="center")

## A simple Rails controller

Let's take a look how this can be used in a Rails controller. Assume we have a controller that should create an RSS feed something like this:

```ruby
class Blog::FeedController < ApplicationController
  def index
    posts = RSS::Posts.new.ordered
    renderer = ApplicationMarkdown.new.renderer
    
    respond_to do |format|
      format.xml { render locals: { posts: posts, renderer: renderer } }
    end
  end
end
```

You can notice the repetition happening at:

```ruby
 { posts: posts, renderer: renderer }
```

Let's do a couple of changes to this code:

1. Removing hash values:
    

```diff
class Blog::FeedController < ApplicationController
  def index
    posts = RSS::Posts.new.ordered
    renderer = ApplicationMarkdown.new.renderer
    
    respond_to do |format|
-      format.xml { render locals: { posts: posts, renderer: renderer } }
+      format.xml { render locals: { posts: , renderer: } }
    end
  end
end
```

We could stop at this change, but I think we can make it a bit more concise or at least play a bit more with the advantage that hash value omission allows the use of methods.

2. Move `posts` and `rendered` to their one methods
    

I am using here the endless method but it will work the same with normal method:

```diff
class Blog::FeedController < ApplicationController
  def index
-    posts = RSS::Posts.new.ordered
-    renderer = ApplicationMarkdown.new.renderer
    
    respond_to do |format|
      format.xml { render locals: { posts: , renderer: } }
    end
  end
  
  private

+    def posts    = RSS::Posts.new.ordered
+    def renderer = ApplicationMarkdown.new.renderer
end
```

Now it will look like this:

```ruby
class Blog::FeedController < ApplicationController
  def index
    respond_to do |format|
      format.xml { render locals: { posts: , renderer: } }
    end
  end
  
  private

    def posts    = RSS::Posts.new.ordered
    def renderer = ApplicationMarkdown.new.renderer
end
```

That for me it is concise enough and has the avdantage that the `index` action is only concerned with the response and composing what is needed for the response while the `posts` and `rendereder` method are creating the necessary objects outside the action because those objects does not depend on the `request/response`

3. Push it a bit more
    

The following change is push the changes a bit too far by extracting also `locals` in a separate method.

```ruby
class Blog::FeedController < ApplicationController
  def index
    respond_to do |format|
      format.xml { render locals: }
    end
  end
  
  private
    def locals   = { posts: , renderer: }
    def posts    = RSS::Posts.new.ordered
    def renderer = ApplicationMarkdown.new.renderer
end
```

This can have the advantage that it isolates the purpose of the `action` method even more about composing the response while delegating the actual composition to the `locals` method. Insofar that if I want to add one more variable to be passed to the response, I don't need any change in the action (which remains focused on composing the response without knowing too much about the internals) and I need only to change the `locals` method.

But it has the disadvantage that is hides those variables in the method so when reading the action `index` it is not clear in a quick glance what will be passed on to the response.

## Guidelines

### Do not mix hash value omission with providing explicit values

I agree here with [Rubocop rule](https://docs.rubocop.org/rubocop/1.65/cops_style.html#enforcedshorthandsyntax-always-default) that either omit value for all hash keys or provide explicit values but don't mix them. It makes reading the code easier for a lot of people. I personally don't have a big issue with reading also mixed style, but I heared a lot of people complain about it.

```ruby
# Do not do this
{ posts:, renderer: renderer }

# Do this
{ posts: posts, renderer: renderer }

# Or do this
{ posts:, renderer: }
```

### Always use paranthesis

There are some edge cases where using the hash value omission might create undesired effects.

Let's assume there are two methods:

```ruby
def bar(x)
  x * 2
end

def foo(a:, b:)
  puts "#{a} and #{b}"
end
```

And now let's test what happens when we try to call `foo` with hash value omission and then run `bar` after it.

```ruby
a = 1
b = 2

puts "Call with parantheses"
foo(a:, b:)
bar(3)
```

The output of this will be:

```bash
Call with parantheses
1 and 2
```

But what if we try to make the calls without paranthesis:

```ruby
a = 1
b = 2

puts "Call without parantheses"
foo a:, b:
bar(3)
```

Can you guess what will be the output?

It will be:

```bash
Call without parantheses
1 and 6
```

This happens because it will execute `b: bar(b)` as in:

```ruby
# The following code 
foo a:, b:
bar(b)

# Is interpreted as being:
foo(a:, b: bar(3))
```

Conclusion: **Always use paranthesis with hash value omission**

## An extra trick

The following trick was [shared](https://bugs.ruby-lang.org/issues/14579#note-26) by Shugo Maeda, the author of the proposal, on the request feature.

Because of how it works you can replace the code that looks like this:

```ruby
def button(class:)
  binding.local_variable_get(:class)
end
```

With code that looks like this:

```ruby
def button(class:)
  { class: }[:class]
end
```

And it is faster too. This benchmark:

```ruby
def access_via_binding(class:)
  binding.local_variable_get(:class)
end

def access_via_hash_value_omission(class:)
  { class: }[:class]
end

Benchmark.ips do |x|
  x.report("method: binding.local_variable_get") do
    access_via_binding(class: :sm)
  end

  x.report("method: hash literal omission") do
    access_via_hash_value_omission(class: :sm)
  end

  x.compare!
end
```

will output the following results on my machine (Macbook M3 Pro):

```bash
ruby 3.3.0 (2023-12-25 revision 5124f9ac75) [arm64-darwin23]
Warming up --------------------------------------
method: binding.local_variable_get
                       532.350k i/100ms
method: hash literal omission
                         1.580M i/100ms
Calculating -------------------------------------
method: binding.local_variable_get
                          5.371M (± 0.6%) i/s -     27.150M in   5.054806s
method: hash literal omission
                         15.776M (± 0.6%) i/s -     79.012M in   5.008478s

Comparison:
method: hash literal omission: 15776220.8 i/s
method: binding.local_variable_get:  5371270.0 i/s - 2.94x  slower
```

## Other resources

If you want to read more about this feature I recommend:

* The release notes of Ruby 3.1: [https://www.ruby-lang.org/en/news/2021/12/25/ruby-3-1-0-released/](https://www.ruby-lang.org/en/news/2021/12/25/ruby-3-1-0-released/)
    
* The Ruby Change log entry about this feature: [https://rubyreferences.github.io/rubychanges/3.1.html#values-in-hash-literals-and-keyword-arguments-can-be-omitted](https://rubyreferences.github.io/rubychanges/3.1.html#values-in-hash-literals-and-keyword-arguments-can-be-omitted)
    
* A deep dive by Victor Shepelev [https://zverok.space/blog/2023-11-10-syntax-sugar3-hash-values-omission.html](https://zverok.space/blog/2023-11-10-syntax-sugar3-hash-values-omission.html)
    
* A critique from Bozhidar Batsov [https://batsov.com/articles/2022/01/20/bad-ruby-hash-value-omission](https://batsov.com/articles/2022/01/20/bad-ruby-hash-value-omission/)
    
* A proposal to adop hash value omission I wrote when I was working at Cookpad to merge it to our style guides: [https://github.com/cookpad/global-style-guides/pull/195](https://github.com/cookpad/global-style-guides/pull/195)
    

---

**Enjoyed this article?**

👐 Subscribe to my Ruby and Ruby on Rails courses over email at [**learn.shortruby.com**](http://learn.shortruby.com/)**\- effortless learning about Ruby anytime, anywhere**

👉 Join my [**Short Ruby Newsletter**](https://newsletter.shortruby.com) for weekly Ruby updates from the community and visit [**rubyandrails.info**](http://rubyandrails.info/)**, a directory with learning content about Ruby.**

🤝 Let's connect on [**Ruby.social**](http://ruby.social/) or [**Linkedin**](https://linkedin.com/in/lucianghinda) or [**Twitter**](https://x.com/lucianghinda) where I post mainly about Ruby and Ruby on Rails

🎥 Follow me on [**my YouTube channel**](https://www.youtube.com/@shortruby) for short videos about Ruby