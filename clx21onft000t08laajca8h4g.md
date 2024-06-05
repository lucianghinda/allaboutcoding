---
title: "Endless method - a quick intro"
seoTitle: "Quick Intro to Ruby Endless Method"
seoDescription: "Quick introduction to the endless method in Ruby, exploring use cases and potential impacts on code structure"
datePublished: Wed Jun 05 2024 16:30:25 GMT+0000 (Coordinated Universal Time)
cuid: clx21onft000t08laajca8h4g
slug: endless-method-a-quick-intro
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1717592767514/6ce3f5d7-881f-4346-84a1-7ef05790f6a9.png
tags: ruby, technology, rails, features, syntax, coding

---

The endless method was added to Ruby version 3.0 in 2020. In a previous article, I wrote about its [history](https://allaboutcoding.ghinda.com/history-of-the-endless-method-syntax) and presented [an example](https://allaboutcoding.ghinda.com/an-endless-method-use-case) of how it was used to refactor a piece of code. Now I am going to explore some use cases.

Here is how it is [described in the Ruby doc](https://docs.ruby-lang.org/en/master/syntax/methods_rdoc.html) (see the yellow part):

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717593076834/db786077-da58-4a32-bf33-1119f14f749e.png align="center")

## Before we begin

Now, let's be clear about the purpose of this discussion. I'm not recommending the endless method as a superior alternative to the normal method, nor am I suggesting that you use it in your code. I want to explore the potential impact on code structure and how we can shape our code using the endless method.

## One-line methods were possible in Ruby before

We could write one-line methods even before endless methods are supported.

The following code is valid Ruby syntax:

```ruby
def valid_array?(array) array.each { |element| validate!(element) } end
def validate!(value) @validator.call(value) end

def name; @name end
```

When looking at the code above let's ask if we really need the `end` keyword to know that for example the `valid_array?` method is finished after the closing bracket `}`. I am asking this visually reading this code.

What if I blurry a bit the `end` keyword. How much does it affect the way you read this code?

![Code sample screenshot with end keyword blurred](https://cdn.hashnode.com/res/hashnode/image/upload/v1717511335432/aa1b1ca1-a1ea-4c1f-b989-de239a20cca5.png align="center")

And here is how this code will look like written with endless method:

```ruby
def valid_array?(array) = array.each { |element| validate!(element) }
def validate!(value) = @validator.call(value)

def name = @name
```

I think there is not too much difference than the example that is a one-line normal Ruby method.

## Endless methods are not required to be one-liners

One thing about the endless method is that it is defined as "single expression" and **not** single line.

Thus we can write - if we want - code like this:

```ruby
def valid?(value) = if value.start_with?("ShortRuby")
  true
else
  false
end
```

Or even code like this:

```ruby
def assignee(organisation, permission:) = case permission[:role]
  when "admin" then Assignee.new(organisation, "admin")
  when "member" then Assignee.new(organisation, "member")
  else Assignee.new(organisation, "guest")
  end
```

Or maybe formatted like this:

```ruby
def assignee(organisation, permission:) = case permission[:role]
when "admin"
  Assignee.new(organisation, "admin")
when "member"
  Assignee.new(organisation, "member")
else
  Assignee.new(organisation, "guest")
end
```

Or maybe like this:

```ruby
def assignee(org, permission:) = case permission[:role]
                                 when "admin"
                                   Assignee.new(org, "admin")
                                 when "member"
                                   Assignee.new(org, "member")
                                 else
                                   Assignee.new(org, "guest")
                                 end
```

## Why use it?

Because using the endless method is cheap (it does not add more lines of code to your file), one of the main advantages I see is to use it to define computed attributes, predicates and in general to give a good name to a statement and thus increase readability.

Here is an example, that is quite common. When writing code to deal with some REST API, we are usually defining a general `get` or `post` methods and then write better (semantically names) for the operations that we plan to use so that when references in code we use the domain name.

Here is an example of a Twitter client where we want to define `me` and `tweets` that wraps a call to `get <url>` so that when this client is used we can write things like `client.tweets` and know that we want to query the tweets.

```ruby
class Twitter::Client
  def me
    get "/users/me"
  end

  def tweets(tweet_ids:)
    get "/tweets?ids=#{tweet_ids.join(',')}"
  end

  private

    def get(path)
      request(:get, path)
    end

    def post(path, body: {}, headers: {})
      request(:post, path, body:, headers:)
    end

    def request(method, path, body: {}, headers: {})
      # ...
    end
end
```

and this could be easily written as:

```ruby
class Twitter::Client
  def me                 = get "/users/me"
  def tweets(tweet_ids:) = get "/tweets?ids=#{tweet_ids.join(',')}"

  private

    def get(path) = request(:get, path)

    def post(path, body: {}, headers: {}) = request(:post, path, body:, headers:)

    def request(method, path, body: {}, headers: {})
      # ...
    end
end
```

## Example in the wild

Here is [Keygen](https://github.com/keygen-sh/keygen-api) - an open, source-available software licensing and distribution API built with Ruby on Rails.

### First example: `Environment` object

I am going to highlight and comment on some pieces of code [extracted](https://github.com/keygen-sh/keygen-api/blob/master/app/models/environment.rb) from their open source app.

```ruby
# Source: https://github.com/keygen-sh/keygen-api/blob/master/app/models/environment.rb

class Environment < ApplicationRecord
  # ... 
  def cache_key    = Environment.cache_key(id, account:)
  def clear_cache! = Environment.clear_cache!(id, code, account:)
  
  def isolated? = isolation_strategy == 'ISOLATED'
  def shared?   = isolation_strategy == 'SHARED'
  
  def admins = account.admins.for_environment(self, strict: false)
  
  def self.codes = reorder(code: :asc).pluck(:code)
  # ...
end
```

In this example we can see a couple of cases for the endless method:

**Defining predicates**

where the endless method helps naming predicated without adding 2 extra lines of code for each name (as it will be the case for a normal method)

```ruby
# Endless method
  def isolated? = isolation_strategy == 'ISOLATED'
  def shared?   = isolation_strategy == 'SHARED'

# vs normal method
  def isolated?
    isolation_strategy == 'ISOLATED'
  end
  
  def shared? 
    isolation_strategy == 'SHARED'
  end
```

I don't see a big readability issue with having an equality comparison (multiple `=` inside the method body) because I read the definition as:

```bash
define isolated as comparing isolation_strategy with ISOLATED
```

**Naming a scoped collection of records**

```ruby
def admins = account.admins.for_environment(self, strict: false)
```

This provides a nice label for a series of scopes chained starting with account. I find it useful to make sure that everywhere in the code where I want the list of admins, I will get the same list and I don't have to remember to add the specific scopes for it.

**Consistent naming**

Further on in the same model we see:

```ruby
  def cache_key    = Environment.cache_key(id, account:)
  def clear_cache! = Environment.clear_cache!(id, code, account:)
```

This is a good DX to attach these commands to a model and thus ask the model what is your cache key or to command it to clear the cache.

### Second example: `LicenseFile` object

Another [example](https://github.com/keygen-sh/keygen-api/blob/master/app/models/license_file.rb) from the same codebase is from the `LienceseFile` object.

```ruby
# Source: https://github.com/keygen-sh/keygen-api/blob/master/app/models/license_file.rb

class LicenseFile
  # ...
  def persisted? = false
  def id         = @id      ||= SecureRandom.uuid
  def product    = @product ||= license&.product
  def owner      = @owner   ||= license&.owner

  def account = @account ||= Account.find_by(id: account_id)
  def account=(account)
    self.account_id = account&.id
  end

  def license = @license ||= License.find_by(id: license_id, account_id:)
  def license=(license)
    self.license_id = license&.id
  end

  # ...
end
```

Allow me to say a couple of things about the endless methods here.

**Memoized simple attributes**

One case for the endless method here is used to load a delegated object:

```ruby
  def id         = @id      ||= SecureRandom.uuid
  def product    = @product ||= license&.product
  def owner      = @owner   ||= license&.owner


  def account = @account ||= Account.find_by(id: account_id)
  # ...
  def license = @license ||= License.find_by(id: license_id, account_id:)
```

**Cohesion**

Another case is keeping code together without wasting space. See here how the getter and setter are close together:

```ruby
  def account = @account ||= Account.find_by(id: account_id)
  def account=(account)
    self.account_id = account&.id
  end
```

It could have been written like:

```ruby
  def account 
    @account ||= Account.find_by(id: account_id)
  end

  def account=(account)
    self.account_id = account&.id
  end
```

But I think it might be something interesting here to write the endless method for the getter just above the setter so then keep them together:

```ruby
  def account = @account ||= Account.find_by(id: account_id)
  def account=(account)
    self.account_id = account&.id
  end

  def license = @license ||= License.find_by(id: license_id, account_id:)
  def license=(license)
    self.license_id = license&.id
  end
```

**Setters cannot be written as endless methods**

You can notice in the example above that the setters are normal methods. That is because the endless method does not support setters. See [this note](https://bugs.ruby-lang.org/issues/16746#note-26) for reference.

## Conclusion

The endless method could be a good fit for simple, one expression methods, when giving a name to some expressions improves readability or consistency. The same can be achieve with a normal method but it will take a bit more vertical space: a normal method takes at least 3 lines for a single one line statement while the endless method can one single line. But of course you can write multi-line endless methods. See the `if` and `case` examples above.

## Recommended reading

Another good introduction was written by Victor Shepelv on [RubyReferences](https://rubyreferences.github.io/rubychanges/3.0.html#endless-method-definition) website and he also wrote [a more in-depth exploration on his blog](https://zverok.space/blog/2023-12-01-syntax-sugar5-endless-methods.html). I strongly recommend you to read them both, specially the in-depth exploration.

---

**Enjoyed this article?**

👐 Subscribe to my Ruby and Ruby on Rails courses over email at [**learn.shortruby.com**](http://learn.shortruby.com/)**\- effortless learning about Ruby anytime, anywhere**

👉 Join my [**Short Ruby Newsletter**](https://newsletter.shortruby.com) for weekly Ruby updates from the community and visit [**rubyandrails.info**](http://rubyandrails.info/)**, a directory with learning content about Ruby.**

🤝 Let's connect on [**Ruby.social**](http://ruby.social/) **or** [**Linkedin**](https://linkedin.com/in/lucianghinda) **or** [**Twitter**](https://x.com/lucianghinda) **where I post mainly about Ruby and Rails.**

🎥 Follow me on [**my YouTube channel**](https://www.youtube.com/@shortruby) for short videos about Ruby