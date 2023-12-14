---
title: "How to create a new Rails app running Rails 7.1 beta or main branch"
seoTitle: "Create a new web app with Rails 7.1"
seoDescription: "How to create a new rails app running Rails 7.1 beta1"
datePublished: Fri Sep 15 2023 10:08:55 GMT+0000 (Coordinated Universal Time)
cuid: clmkft5kk000309l5daafh7xy
slug: how-to-create-a-new-rails-app-running-rails-71-beta-or-main-branch
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1694772438809/911c019d-e1b2-4a69-8623-4055a47dd407.png
tags: newbie, news, ruby, rails

---

### New app with Rails 7.1.beta1

Here is how to create a new Rails app that runs on Rails 7.1 beta1

```bash
gem install -v 7.1.0.beta1 rails

rails _7.1.0.beta1_ new myrails71app
```

Replace `myrails71app` with your own app name and you are good to go.

This will generate a `Gemfile` that has something like this inside and then install required gems for `Rails 7.1.0.beta1`

```ruby
# Gemfile

gem "rails", "~> 7.1.0.beta1"
```

### New app with Rails main branch

Here is how to create a new Rails app that runs on Rails main branch from GitHub:

```bash
rails new myrailsapp --main
```

This will generate a `Gemfile` that has something like this inside:

```ruby
gem "rails", github: "rails/rails", branch: "main"
```

May you have a lot of ideas to try!

---

Enjoyed this article?

Join my [**Short Ruby News**](https://shortruby.com/) **newsletter** for weekly Ruby updates from the community. For more Ruby **learning resources**, visit [**rubyandrails.info**](http://rubyandrails.info). You can also find me on [**Ruby.social**](https://ruby.social/@lucian) or [**Linkedin**](https://linkedin.com/in/lucianghinda) or [**Twitter**](https://x.com/lucianghinda) where I post mostly about Ruby and Rails. Subscribe to [**my YouTube channel**](https://www.youtube.com/@shortruby) for short videos about Ruby.