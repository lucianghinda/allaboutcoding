---
title: "Ruby Open Source: Zammad example"
seoTitle: "Zammad Ruby on Rails Open Source Example"
seoDescription: "Explore Zammad: open-source Ruby ticketing system with cloud, custom styling, multiple databases, and service objects on Ruby 3.1.3, Rails 7"
datePublished: Fri Nov 03 2023 03:52:55 GMT+0000 (Coordinated Universal Time)
cuid: cloi2yd2y000c09jtgkw8ceoe
slug: ruby-open-source-zammad-example
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1698983459643/94aeb1b5-df0d-48b1-a725-e14dfb466096.png
tags: programming-blogs, ruby, opensource, ruby-on-rails, coding

---

I will probably start a series of open-source Ruby Projects. Maybe I will call it #opensource #Friday.

[Zammad](https://zammad.com/en) is an open-source ticketing system, that also offers an on-cloud product.

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697602812644/7f4332e3-f201-4b13-9665-edcc4b62ba93.png align="center")](https://zammad.com/en)

They have their product open-sourced on [Github](https://github.com/zammad/zammad) and it is built using (at the moment of writing this article) **Ruby 3.1.3** and **Rails 7**

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697602714959/cb56b30a-549e-425b-b779-aa6d0c4439fc.png align="center")](https://github.com/zammad/zammad)

## Licensing

The license is [GNU AGPL](https://github.com/zammad/zammad/blob/develop/LICENSE)

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697612651893/a97191ad-5d59-4f61-b9f4-d310eda87e2d.png align="center")](https://github.com/zammad/zammad/blob/develop/LICENSE)

They [mention on their website](https://zammad.com/en/company/open-source) why they chose to make the product open source:

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697612717159/df787418-80d9-4629-9b45-12104b3fca55.png align="center")](https://zammad.com/en/company/open-source)

Each file in the repository has a license line like:

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697613596157/626b9c49-9262-47c2-a3b2-144bf95f9728.png align="center")](https://github.com/zammad/zammad/blob/develop/Gemfile)

## Some ideas from the open-source repo

I don't have enough time to analyse in depth the repo. So just looking around for 30 minutes, here are some things that I extracted

### Stats

Running `rails stats` returned the following:

![Result of executing rails stats on the repo](https://cdn.hashnode.com/res/hashnode/image/upload/v1698983056409/424bdfb1-3437-4fb2-a49a-6e421b6066b4.png align="center")

### Styling Guide

They use Rubocop with extra cops added. They require some cops provided by gems and custom cops written by them:

```ruby
require:
  - rubocop-capybara
  - rubocop-factory_bot
  - rubocop-faker
  - rubocop-graphql
  - rubocop-inflector
  - rubocop-performance
  - rubocop-rails
  - rubocop-rspec
  - ../config/initializers/inflections.rb
  - ./rubocop_zammad.rb
```

In `rubocop_zammad.rb` they are loading custom cops from [`.rubocop/cop/zammad`](https://github.com/zammad/zammad/tree/develop/.rubocop/cop/zammad)

Here are *some* custom cops written by them:

* [Checking](https://github.com/zammad/zammad/blob/develop/.rubocop/cop/zammad/correct_migration_timestamp.rb) if the migration file starts with a valid timestamp
    
* [Checking](https://github.com/zammad/zammad/blob/develop/.rubocop/cop/zammad/exists_condition.rb) usages of `find_by` to check if an Active Record exists and replace it with `exists?`
    
* [Checking](https://github.com/zammad/zammad/blob/develop/.rubocop/cop/zammad/forbid_default_scope.rb) that the only allowed `default_scope` is about simple ordering
    
* [Checking](https://github.com/zammad/zammad/blob/develop/.rubocop/cop/zammad/forbid_rand.rb) for using `rand`
    
* [Checking](https://github.com/zammad/zammad/blob/develop/.rubocop/cop/zammad/no_to_sym_on_string.rb) usages of `.to_sym` on strings and change to `:` prefix
    
* [Checking](https://github.com/zammad/zammad/blob/develop/.rubocop/cop/zammad/prefer_negated_if_over_unless.rb) usages of `unless` and suggest using `!`
    
* [Checking](https://github.com/zammad/zammad/blob/develop/.rubocop/cop/zammad/update_copyright.rb) copyright notice and adding it when missing
    

More about their style guide can be found at [`doc/developer_manual/standards`](https://github.com/zammad/zammad/blob/develop/doc/developer_manual/standards/code-style-guide.md)

### Persistence

They appear to use 3 DBs, each one having their group. One ([the activerecord-nulldb-adapter](https://github.com/nulldb/nulldb)) is actually a NullObject pattern implemented for Active Record.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697613005230/e9580f24-8d99-4bf4-a9cd-df14a0726516.png align="center")

Based on the loaded connection they do a preflight check [in an initializer](https://github.com/zammad/zammad/blob/develop/config/initializers/db_preflight_check.rb). This is what it looks like:

```ruby
Rails.application.config.after_initialize do
  Zammad::Application::Initializer::DbPreflightCheck.perform
end
```

and you can go check [`lib/zammad/application/initializer`](https://github.com/zammad/zammad/tree/develop/lib/zammad/application/initializer/db_preflight_check) to see what kind of checks are executed for each adapter.

This is a way to make sure that the actual DB server respects a contract they have defined in these initializers (e.g. what extensions are activated or config defaults or minimum version).

See this example of a check for MySQL from the same file:

![Code sample from MySQL DB preflight check](https://cdn.hashnode.com/res/hashnode/image/upload/v1698982343291/90dad251-2dda-44dc-86e4-842e72de559e.png align="center")

### Some gems used

* [argon2](https://github.com/technion/ruby-argon2) - *"A Ruby gem offering bindings for Argon2 password hashing"*
    
* [rszr](https://github.com/mtgrosser/rszr) - "*Rszr is an image resizer for Ruby based on the Imlib2 library. It is faster and consumes less memory than MiniMagick, GD2 and VIPS, and comes with an optional drop-in interface for Rails ActiveStorage image processing*"
    
* [biz](https://github.com/zendesk/biz) - *"Time calculations using business hours"*
    
* [diffy](https://github.com/samg/diffy) - *"It provides a convenient way to generate a diff from two strings or files. Instead of reimplementing the LCS diff algorithm Diffy uses battle tested Unix diff to generate diffs, and focuses on providing a convenient interface, and getting out of your way"*
    
* [chunky\_png](https://github.com/wvanbergen/chunky_png) - *"ChunkyPNG is a pure Ruby library to read and write PNG images and access textual metadata. It has no dependency on RMagick, or any other library for that matter"*
    
* [localhost](https://github.com/socketry/localhost) - *"This gem provides a convenient API for generating per-user self-signed root certificates"*
    
* [activerecord-nulldb](https://github.com/nulldb/nulldb) - "*NullDB is the Null Object pattern as applied to ActiveRecord database adapters. It is a database backend that translates database interactions into no-ops. Using NullDB enables you to test your model business logic - including after\_save hooks - without ever touching a real database"*
    

### Design Patterns

They use **service objects**. There is a [`Service::Base`](https://github.com/zammad/zammad/blob/develop/app/services/service/base.rb) class that is empty and then there is also [`Service::BaseWithCurrentUser`](https://github.com/zammad/zammad/blob/develop/app/services/service/base_with_current_user.rb) that looks something like this:

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697617174559/c087e526-1895-4740-b0a0-261223e24d47.png align="center")](https://github.com/zammad/zammad/blob/develop/app/services/service/base_with_current_user.rb)

The main method that a Service object should define is `execute` but there is no enforcement of this. It is just that everything under [app/services/service](https://github.com/zammad/zammad/tree/develop/app/services/service) has this method defined.

**GraphQL objects**

All the logic about GraphQL is inside [app/graphql](https://github.com/zammad/zammad/tree/develop/app/graphql), namescoped to `Gql.`

**Jobs**

They can be found at [app/jobs](https://github.com/zammad/zammad/tree/develop/app/jobs)**.** Job priority is defined in a concern called [`ApplicationJob::HasQueuingPriorit`](https://github.com/zammad/zammad/blob/develop/app/jobs/application_job/has_queuing_priority.rb)`y` that looks like this:

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697617529027/969eb9ec-d08d-47ed-9877-7a8d8a5c7b9b.png align="center")](https://github.com/zammad/zammad/blob/develop/app/jobs/application_job/has_queuing_priority.rb)

And all jobs have a default priority of **200** while low\_priority is defined as being **300**

**Models**

They are under [app/models](https://github.com/zammad/zammad/tree/develop/app/models) and there is an [`ApplicationModel`](https://github.com/zammad/zammad/blob/develop/app/models/application_model.rb) defined that includes some defaults like [this](https://github.com/zammad/zammad/blob/develop/app/models/application_model.rb):

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697617801805/e23db320-1e38-4684-b485-2e42f50ec466.png align="center")](https://github.com/zammad/zammad/blob/develop/app/models/application_model.rb)

There are probably a lot more interesting things to discover about the codebase but this is what I got in a short review.

---

Enjoyed this article?

Join my [**Short Ruby News**](https://shortruby.com/) **newsletter** for weekly Ruby updates from the community. For more Ruby **learning resources**, visit [**rubyandrails.info**](http://rubyandrails.info). You can also find me on [**Ruby.social**](https://ruby.social/@lucian) or [**Linkedin**](https://linkedin.com/in/lucianghinda) or [**Twitter**](https://x.com/lucianghinda) where I post mostly about Ruby and Rails. Subscribe to [**my YouTube channel**](https://www.youtube.com/@shortruby) for short videos about Ruby.