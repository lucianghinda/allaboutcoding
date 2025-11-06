---
title: "RSpec and `let!`: Understanding the Potential Pitfalls"
seoTitle: "Avoiding RSpec `let!` Pitfalls"
seoDescription: "Avoid using `let!` in RSpec tests and discover alternative approaches to increase test readability and debugging"
datePublished: Thu Nov 06 2025 08:27:35 GMT+0000 (Coordinated Universal Time)
cuid: cmhn5ytz8000102kzhi3d1my3
slug: rspec-and-let-understanding-the-potential-pitfalls
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1762417577219/8f86cf1b-8073-42b3-8cb8-176b6a1a84a4.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1762417617860/6f433cb6-b4b4-4c01-8b7a-db0a7521d6e7.png
tags: ruby, ruby-on-rails, testing, rspec

---

This is not a new topic; various resources have addressed it in different ways. Here are my reasons and explanations for why I prefer not to use 'let!' in RSpec.

When I work on a project that uses RSpec, I prefer not to use `let!`. Instead, I call the let variable inside the `before` block.

```ruby
RSpec.describe Thing do   
 let(:precondition) { create(:item) }

 before  
   precondition  
 end

 it 'returns that specific value' do  
   # do
   # expect  
 end  
end
```

Taking it a step further, if you do not need to reference `precondition` in your tests, you can do this instead:

```ruby
RSpec.describe Thing do   
 before  
   create(:item)  
 end

 it 'returns that specific value' do  
   # do
   # expect  
 end  
end
```

First what does `let!` do? It just [a call](https://github.com/rspec/rspec/blob/main/rspec-core/lib/rspec/core/memoized_helpers.rb#L329) to the normal `let` and then setting a `before` block for the same name.

```ruby
# source: https://github.com/rspec/rspec/blob/main/rspec-core/lib/rspec/core/memoized_helpers.rb#L329
def let!(name, &block)
  let(name, &block)
  before { __send__(name) }
end
```

## Reason 1: `let!` is actually a precondition for a test and it hides it

In testing `let!` acts as a test precondition: the required state of the system before running a specific test. This should be visible while reading the test and using `!` for this makes it hard to spot and hard to load it in the mental context.

It is harder to notice a `let!` among several `let` statements when you are trying to debug than seeing a `before` block which clearly communicate intention to run some preconditions.

```ruby
RSpec.describe Thing do   
 let(:account) { build(:account) }  
 let!(:organisation) { build(:organisation, account: account) }

 it 'returns that specific value that we want' do  
   # test  
 end  
end
```

compared with:

```ruby
RSpec.describe Thing do
 let(:account) { build(:account) }  
 let(:organisation) { build(:organisation, account: account) }

 before
   organisation  
 end

 it 'returns that specific value that we want' do  
    # test 
 end  
end
```

Which version makes it clearer that the organisation is created before each test?

## Reason 2: When needed, the order of creating the objects is easily visible

Here is a comparison:

```ruby
RSpec.describe Thing do   
 let!(:account_a) { build(:user, email: email) }  
 let!(:account_b) { build(:user, user: email2) }  
 let(:organisation) { build(:organisation, account: account) }  
 let(:team) { build(:team, account: organisation) }  

 it 'returns that specific value that we want' do  
    # test 
 end  
end
```

compared to

```ruby
RSpec.describe Thing do   
 let(:account_a) { build(:user, email: email) }  
 let(:account_b) { build(:user, user: email2) }  
 let(:organisation) { build(:organisation, account: account) }  
 let(:team) { build(:team, account: organisation) }  

 before  
   account_a  
   account_b  
 end

 it 'returns that specific value that we want' do  
    # test 
 end  
end
```

But imagine that after a series of changes there is a risk that someone might put a `let!` among other calls.

```ruby
RSpec.describe Thing do   
 let!(:account_a) { build(:user, email: email) }  
 let(:organisation) { build(:organisation, account: account) }  
 let!(:account_b) { build(:user, user: email2) }
 let(:team) { build(:team, account: organisation) }  

 it 'returns that specific value that we want' do  
    # test 
 end  
end
```

## A Minitest equivalent

Here are three ways to write something similar in Minitest:

```ruby
class ThingTest < Minitest::Test  
   def setup  
       @account_a = build(:user, email: email)  
       @account_b = build(:account, user: user)  
       @organisation =  build(:organisation, account: account)  
   end

   def test_computed_slug_returns_with_dashes  
       @account_a.name = "My Name"        
  
       assert_equal "my-name", @account_a.computed_slug  
   end  
end
```

Here is another approach:

```ruby
class ThingTest < Minitest::Test  
   attr_accessor :account_a, :account_b, :organisation  
     
   def setup  
       @account_a = build(:user, email: email)  
       @account_b = build(:account, user: user)  
       @organisation =  build(:organisation, account: account)  
   end

   def test_computed_slug_returns_with_dashes  
       account_a.name = "My Name"

       assert_equal "my-name", account_a.computed_slug  
   end  
end
```

Since this is plain Ruby, you can also do the following:

```ruby
class ThingTest < Minitest::Test  
   def test_computed_slug_returns_with_dashes  
       account_a.name = "My Name"

       assert_equal "my-name", account_a.computed_slug  
   end 
   
   private 
   
   def account_a    = build(:user, email: email)  
   def account_b    = build(:account, user: user)  
   def organisation =  build(:organisation, account: account)  
end
```

---

👉 If you like this article and want it in your inbox each week, [subscribe to my newsletter](https://newsletter.lucianghinda.com). You’ll find **ideas on Ruby, software development, software testing, building products and workshops**, plus notes on creativity, tech trends, and whatever else sparks my curiosity.

👐 Want to improve your **developer testing skills**? Visit [goodenoughtesting.com/articles](https://goodenoughtesting.com/articles) to discover resources on testing for developers.

👉 [Join my Short Ruby Newsletter](https://newsletter.shortruby.com) for weekly Ruby updates and visit rubyandrails.info, a directory of Ruby learning content.

🤝 Connect with me on [Linkedin](https://linkedin.com/in/lucianghinda), [Bluesky](https://bsky.app/profile/lucianghinda.com), [Ruby.social](https://ruby.social/@lucian), , and [Twitter](https://x.com/lucianghinda), where I mostly post about Ruby and Ruby on Rails.

🎥 Follow [my YouTube channel](https://www.youtube.com/@shortruby) for short videos about Ruby and Rails.