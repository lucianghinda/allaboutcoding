---
title: "Refactoring transaction blocks with the endless method"
seoTitle: "Transactions blocks with endless methods"
seoDescription: "Explore refactoring Rails transaction blocks with the endless method. Pros and cons, examples from Maybe and Mastodon, and community feedback"
datePublished: Thu Jun 20 2024 06:49:09 GMT+0000 (Coordinated Universal Time)
cuid: clxmwixh7000u0al12expayt6
slug: refactoring-transaction-blocks-with-the-endless-method
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1718866028564/ea0c5374-c24f-4986-984c-976aad70b2b9.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1718866055049/8af49170-c67a-4c9f-8483-478439b51861.png
tags: refactoring, programming-blogs, ruby, ruby-on-rails

---

You can use the endless method to name a transaction block in Rails.

This is a technique **Kasper Timm Hansen**[shared](https://ruby.social/@kaspth/112571908837369243) in a reply to my previous post. I will try to refactor some examples from two open-source Rails repositories just to explore how the code looks.

This open-ended exercise is a playground for experimenting with the code shape and seeing if it offers any advantages. Your feedback is not only welcome but anticipated. I invite you to play with it, too, and try to assess from various perspectives how it affects your ability to read and understand the code.

I will present two examples from two open-source Rails apps. I didn't submit these refactorings as PRs to them since they are an aesthetic change and are not functional. There is no need to waste maintainers' time reviewing them.

## Refactoring an example from Maybe source code

Here is an example from [Maybe open source Rails](https://github.com/maybe-finance/maybe) app where a method is defined on an Active Record model.

```ruby
# Source: https://github.com/maybe-finance/maybe/blob/main/app/models/transaction/category.rb
class Transaction::Category < ApplicationRecord
  # ...
  def replace_and_destroy!(replacement)
    transaction do
      transactions.update_all category_id: replacement&.id
      destroy!
    end
  end
  # ...
end
```

It seems to me to be a nice, short method, having all code wrapped into a transaction thus a code candidate for refactoring with the technique proposed by Kasper.

Here is a proposal about how to refactor it to an endless multi-line method:

```diff
# Source: https://github.com/maybe-finance/maybe/blob/main/app/models/transaction/category.rb
class Transaction::Category < ApplicationRecord
  # ...
-  def replace_and_destroy!(replacement)
-    transaction do
+  def replace_and_destroy!(replacement) = transaction do
      transactions.update_all category_id: replacement&.id
      destroy!
-    end
  end
  # ...
end
```

Here's how the final code might look: the \`transaction\` on the same line as the method definition, with the block nested below.

```ruby
# Source: https://github.com/maybe-finance/maybe/blob/main/app/models/transaction/category.rb
class Transaction::Category < ApplicationRecord
  # ...
  def replace_and_destroy!(replacement) = transaction do
    transactions.update_all category_id: replacement&.id
    destroy!
  end
  # ...
end
```

## Refactoring an example from Mastodon source code

Let's take a look at another example - this time from [Mastodon](https://github.com/mastodon/mastodon) source code Rails app. The focus here will be to extract the lines 7-10 into their own method and give them a (hopefully) good name:

```ruby
# Source: https://github.com/mastodon/mastodon/blob/main/app/services/keys/claim_service.rb#L37

def claim_local_key!
  device = @target_account.devices.find_by(device_id: @device_id)
  key    = nil

  ApplicationRecord.transaction do
    key = device.one_time_keys.order(Arel.sql('random()')).first!
    key.destroy!
  end

  @result = Result.new(@target_account, @device_id, key)
end
```

First I will replace all those lines with a name that I think might be good enough to describe the outcome of that block (probably I don't have enough domain knowledge from Mastodon codebase to find a better one so far)

```diff
# Source: https://github.com/mastodon/mastodon/blob/main/app/services/keys/claim_service.rb#L37

def claim_local_key!
  device = @target_account.devices.find_by(device_id: @device_id)
-  key    = nil
-
-  ApplicationRecord.transaction do
-    key = device.one_time_keys.order(Arel.sql('random()')).first!
-    key.destroy!
-  end
+  key = existing_key(device)

  @result = Result.new(@target_account, @device_id, key)
end
```

And then I created a new method using the same technique I used for Maybe example:

```ruby
def existing_key(device) = transaction do 
  key = device.one_time_keys.order(Arel.sql('random()')).first!
  key.destroy!
end
```

A multi line endless method with \`transaction\` on the same line as the method definition and the block content nested below.

So the final code looks something like this:

```ruby
def claim_local_key!
  device = @target_account.devices.find_by(device_id: @device_id)
  key = existing_key(device)

  @result = Result.new(@target_account, @device_id, key)
end

def existing_key(device) = transaction do 
  key = device.one_time_keys.order(Arel.sql('random()')).first!
  key.destroy!
end
```

## Reasons in favour of this refactoring

### Removes an extra nesting level

Usually when there is a method that has inside a block visually there is a nested `do/end` and using this technique you can reduce that. In these examples the nesting is only one level deep so maybe there is not much advantage gained by doing this. But could be helpful in situations where there are multiple nested blocks.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718858631120/07a9adf0-48ca-4981-8230-eb90aae4d873.png align="center")

### Signaling that everything in that method is a transaction

Another possible reason is that it makes it clear that everything in that method is inside a transaction block and it makes it clear from the start - being on the same line as method definition:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718858688887/3c266a10-db6f-440c-856a-841bedac0b51.png align="center")

## Reasons against this refactoring

### Hides complexity and makes the reading non-linear

Sometimes imperative code (or steb by step explicite code) is easier to read and thus maybe extracting this code to a method will hide the fact that there is a transaction happening there that deletes a previous key.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718858763040/39c244d0-b540-4179-8738-9795ec0c24dc.png align="center")

### Skipping method definition could mean missing the transaction

Depending on how you read or skim Ruby code if you skip method definition line and go directly to the body you might miss the `transaction` keyword there:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718858812350/c740c748-c34f-45e4-b6cb-0988ecff9db4.png align="center")

## Reviews

I posted this on social media on [Twitter](https://x.com/lucianghinda/status/1803367120790843413) and [Mastodon](https://ruby.social/@lucian/112642700200514809) and got a wide range of feedback about it. I think there are good arguments so I will try to summarise some of them here with some of my thoughts about them, with a focus on the ones that are providing a critique of this technique.

### Hard to read method definition

[Gauthier Monserand](https://ruby.social/@gmonserand@mastodon.social) shared [in a reply](https://ruby.social/@gmonserand@mastodon.social/112643119072292941) that they find hard to read a sequence like `def name = transaction do` or the endless method in general `def name = body`.

On the same direction that having important information after the method definition can be easy to miss is the feedback [shared](https://x.com/jakeonrails/status/1803632444068733247) by [Jake Moffatt](https://x.com/jakeonrails):

> While reading the thread this is the thing that was nagging at me the whole time. It's unexpected and easy to miss a hidden method call after the method declaration. Personally I don't feel it increases clarity, nor is one level of indentation worth potential misunderstanding

I think this is valid point. When I encounter a new code shape I have the same reaction, I have to read it multiple times. But I also think sometimes this point has more to do with familiarity and not the code shape on its own. I am referring here to a language construct/expression and not methods that are hard to read because they are packing too much obscure language features into them.

[Russell Garner](https://ruby.social/@rgarner@mastodon.social) had a very good argument when they [say](https://ruby.social/@rgarner@mastodon.social/112642839681091879):

> Sometimes we establish convention by determining that something is an improvement in concrete ways.
> 
> And sometimes we fail to establish convention purely because it's unconventional, nobody does it like that and hence nobody can read it reliably.

### Making the code non-linear

[Dima Fatko](https://x.com/fatkodima)[mentioned](https://x.com/fatkodima/status/1803388532192329844) that doing this kind of transformation make the code "strange, cryptic and non-linear flow".

> Previously, the code was simple, clear, idiomatic, had linear flow. Now, the code is strange, cryptic, non-linear flow - you need to look past the method definition to see that there is some other code, wat

I think for the entire process of refactoring that I did Dima has a point. In general the price to pay for naming things can go along the lines of non-linear reading of the code. Add to this mix using a less known feature of the language can make the code a bit harder to read. Extracting and naming cost is indeed one that has to be decided by the developer or the team and as Dima noticed it does not have a clear winner.

The part about less known code is fixable by trying to use this new syntax. Once you start reading endless methods you get in the habit to look past method name to the left to get some meaning. For example Elixir has guard clauses that looks like this:

```elixir
def my_function(number) when is_integer(number) and rem(number, 2) == 0 do
  # do stuff
end
```

I am not saying Elixir is better or worse but the fact that there are programmers writing and reading code like this is an argument that important information can live on the same line as method definition and the code remains readable and maintainable to them.

### Packing more into the method definition

This is a very intersting concept on its own [shared](https://ruby.social/@soulcutter/112643351538307875) by [Bradley Schaefer](https://ruby.social/@soulcutter) about the idea to pack more info into the method definition. He proposed to add also the private method in front of the code and it will probably make it look like this:

```ruby

private def replace_and_destroy!(replacement) = transaction do
  transactions.update_all category_id: replacement&.id
  destroy!
end

# or maybe like this

private def replace_and_destroy!(replacement) = transaction do
          transactions.update_all category_id: replacement&.id
          destroy!
        end
```

I like this idea to pack more high-level context or constraints around the method definition.

### Using endless methods to organize them in "dense blocks"

This is another argument from [Bradley Schaefer](https://ruby.social/@soulcutter) shared [on Lobsters](https://lobste.rs/s/nfoya5/useless_ruby_sugar_endless_one_line#c_kx2xqu) that I think it is worth sharing here on its entirety:

![Screenshot of a text shared on Lobsters about using the endless method](https://cdn.hashnode.com/res/hashnode/image/upload/v1718861228277/52608abe-e418-4f67-8189-c6b8269473a4.png align="center")

### Add the most important piece of information before the method definition

[**Jake Moffatt**](https://x.com/jakeonrails) proposed a better way to revel the `transaction` information by making it explicit and preceding the definition of a normal method:

```ruby
transaction def replace_and_destroy!(replacement)
  transactions.update_all category_id: replacement&.id
  destroy!
end
```

I really like this idea and if would not be for the price to pay for this - adding metaprogramming - that he mentions I think we would see more code that brings some important context as a kind of prefix for the method call.

I also agree to some extend with [**Mikael Henriksson**](https://x.com/mhenrixon)**'s**[post](https://x.com/mhenrixon/status/1803666199366058435) that *"method macros can make the code hard to reason about"*. And I would split the method macros in two: part of the language and created by the user. I think `private` or `protected` keywords can be safely added to a method and would not affect the readability/understandability of the method, but when defining custom macros than it indeed a balance to consider and a choice to make.

---

**Enjoyed this article?**

👐 Subscribe to my Ruby and Ruby on Rails courses over email at [**learn.shortruby.com**](http://learn.shortruby.com/)**\- effortless learning about Ruby anytime, anywhere**

👉 Join my [**Short Ruby Newsletter**](https://newsletter.shortruby.com) for weekly Ruby updates from the community and visit [**rubyandrails.info**](http://rubyandrails.info/)**, a directory with learning content about Ruby.**

🤝 Let's connect on [**Ruby.social**](http://ruby.social/) or [**Linkedin**](https://linkedin.com/in/lucianghinda) or [**Twitter**](https://x.com/lucianghinda) where I post mainly about Ruby and Ruby on Rails

🎥 Follow me on [**my YouTube channel**](https://www.youtube.com/@shortruby) for short videos about Ruby