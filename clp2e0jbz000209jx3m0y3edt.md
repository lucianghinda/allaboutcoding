---
title: "Ruby open source: feedbin"
seoTitle: "Feedbin: Ruby Open Source RSS Reader"
seoDescription: "Explore Feedbin, an open-source Ruby on Rails platform combining RSS feeds with newsletters so that you can enjoy content on web"
datePublished: Fri Nov 17 2023 08:57:56 GMT+0000 (Coordinated Universal Time)
cuid: clp2e0jbz000209jx3m0y3edt
slug: ruby-open-source-feedbin
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1700119795610/ad6866be-40ac-42d7-baa1-47cef8181b38.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1700119929433/d2fc47fb-8ed6-42f7-8620-7a091899c8ec.png
tags: programming-blogs, ruby, opensource, ruby-on-rails, coding

---

## The product

[https://feedbin.com](https://feedbin.com)

> "Feedbin is the best way to enjoy content on the Web. By combining RSS, and newsletters, you can get all the good parts of the Web in one convenient location"

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699934509194/dcaac2f3-34e3-4dad-8125-12cc20aa51a9.png align="center")](https://feedbin.com/about)

## Open source

The open-source repository can be found at [https://github.com/feedbin/feedbin](https://github.com/feedbin/feedbin/blob/main/LICENSE.md)

### License

The [license](https://github.com/feedbin/feedbin/blob/main/LICENSE.md) they use is MIT:

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699934721882/26c3731c-c7c1-4d6b-833f-2f394834e360.png align="center")](https://github.com/feedbin/feedbin/blob/main/LICENSE.md)

## Technical review

### Ruby and Rails version

They are currently using:

* Ruby version 3.2.2
    
* They used a fork of Rails at [https://github.com/feedbin/rails](https://github.com/feedbin/rails) forked from [https://github.com/Shopify/rails](https://github.com/Shopify/rails). They are using a branch called [7-1-stable-invalid-cache-entries](https://github.com/feedbin/rails/tree/7-1-stable-invalid-cache-entries) - It seems to be Rails 7.1 and about 1 month behind the Shopify/rails which is usually pretty up to date with main Rails
    

### Architecture

Code Architecture:

* They are using the standard Rails organisation of MVC.
    

Database:

* The DB is PostgreSQL
    

Jobs queue:

* Sidekiq
    

On the front-end side:

* They use `.html.erb`
    
* They are using Phlex for [components](https://github.com/feedbin/feedbin/tree/main/app/views/components)
    
* They are using [Jquery](https://github.com/feedbin/feedbin/blob/main/Gemfile#L38) for the JS library
    
* They have some custom JS code written in [CoffeeScript](https://github.com/feedbin/feedbin/tree/main/app/assets/javascripts/web)
    
* They are using Hotwire via [importmaps](https://github.com/feedbin/feedbin/blob/abf1ad883dab8a3464fe12e4653de6323296175b/config/importmap.rb#L1)
    
* They are using [Tailwind](https://github.com/feedbin/feedbin/blob/abf1ad883dab8a3464fe12e4653de6323296175b/Gemfile#L66)
    

### Stats

Running `/bin/rails stats` will output the following:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699935469149/12169b38-413f-43c4-a9ae-08cbf61e2db7.png align="center")

Running VSCodeCounter will give the following stats:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699935527677/99f9ce55-540f-452f-a585-cea43fd8dcd4.png align="center")

### Style Guide

For Ruby:

* They are using [standardrb](https://github.com/feedbin/feedbin/blob/abf1ad883dab8a3464fe12e4653de6323296175b/Gemfile#L94) as the Style Guide with no customisations.
    

### Storage, Persistence and in-memory storage

The DB is PostgreSQL.

They are not using the schema.rb but the [structure.sql](https://github.com/feedbin/feedbin/blob/main/db/structure.sql) format for DB schema dump is configured via application.rb:

```ruby
module Feedbin
    class Application < Rails::Application
        # other configs
        config.active_record.schema_format = :sql
        # other configs
    end
end
```

Enabled PSQL extensions:

* hstore - "data type for storing sets of (key, value) pairs"
    
* pg\_stat\_statements - "track planning and execution statistics of all SQL statements executed"
    
* uuid-ossp - "generate universally unique identifiers (UUIDs)"
    

```sql
CREATE EXTENSION IF NOT EXISTS hstore WITH SCHEMA public;
COMMENT ON EXTENSION hstore IS 'data type for storing sets of (key, value) pairs';

CREATE EXTENSION IF NOT EXISTS pg_stat_statements WITH SCHEMA public;
COMMENT ON EXTENSION pg_stat_statements IS 'track planning and execution statistics of all SQL statements executed';


CREATE EXTENSION IF NOT EXISTS "uuid-ossp" WITH SCHEMA public;
COMMENT ON EXTENSION "uuid-ossp" IS 'generate universally unique identifiers (UUIDs)';
```

Redis is configured to be used with Sidekiq.

This is what the [redis initializer](https://github.com/feedbin/feedbin/blob/abf1ad883dab8a3464fe12e4653de6323296175b/config/initializers/redis.rb#L1) looks like:

```ruby
# https://github.com/feedbin/feedbin/blob/main/config/initializers/redis.rb#L1

defaults = {connect_timeout: 5, timeout: 5}
defaults[:url] = ENV["REDIS_URL"] if ENV["REDIS_URL"]

$redis = {}.tap do |hash|
  options2 = defaults.dup
  if ENV["REDIS_URL_PUBLIC_IDS"] || ENV["REDIS_URL_CACHE"]
    options2[:url] = ENV["REDIS_URL_PUBLIC_IDS"] || ENV["REDIS_URL_CACHE"]
  end
  hash[:refresher] = ConnectionPool.new(size: 10) { Redis.new(options2) }
end
```

Further, there is a [RedisLock](https://github.com/feedbin/feedbin/blob/main/app/models/redis_lock.rb#L1) configured like this:

```ruby
# https://github.com/feedbin/feedbin/blob/main/app/models/redis_lock.rb#L1

class RedisLock
  def self.acquire(lock_name, expiration_in_seconds = 55)
    Sidekiq.redis { _1.set(lock_name, "locked", ex: expiration_in_seconds, nx: true) }
  end
end
```

Further down this is used in a [clock.rb](https://github.com/feedbin/feedbin/blob/main/lib/clock.rb#L8) (that defines scheduled tasks to run):

```ruby
# https://github.com/feedbin/feedbin/blob/main/lib/clock.rb#L8

every(10.seconds, "clockwork.very_frequent") do
  if RedisLock.acquire("clockwork:send_stats:v3", 8)
    SendStats.perform_async
  end

  if RedisLock.acquire("clockwork:cache_entry_views", 8)
    CacheEntryViews.perform_async(nil, true)
  end

  if RedisLock.acquire("clockwork:downloader_migration", 8)
    FeedCrawler::PersistCrawlData.perform_async
  end
end

every(1.minutes, "clockwork.frequent") do
  if RedisLock.acquire("clockwork:feed:refresher:scheduler:v2")
    FeedCrawler::ScheduleAll.perform_async
  end

  if RedisLock.acquire("clockwork:harvest:embed:data")
    HarvestEmbeds.perform_async(nil, true)
  end
end

every(1.day, "clockwork.daily", at: "7:00", tz: "UTC") do
  if RedisLock.acquire("clockwork:delete_entries:v2")
    EntryDeleterScheduler.perform_async
  end

  if RedisLock.acquire("clockwork:trial_expiration:v2")
    TrialExpiration.perform_async
  end

  if RedisLock.acquire("clockwork:web_sub_maintenance")
    WebSub::Maintenance.perform_async
  end
end
```

### Gems used

Here are some of the gems used:

* [sax-machine](https://github.com/pauldix/sax-machine) - "A declarative sax parsing library backed by Nokogiri"
    
* [feedjira](https://github.com/feedjira/feedjira) - "Feedjira is a Ruby library designed to parse feeds"
    
* [html-pipeline](https://github.com/feedbin/html-pipeline) - "HTML processing filters and utilities. This module is a small framework for defining CSS-based content filters and applying them to user provided content"
    
* [apnotic](https://github.com/ostinelli/apnotic) - "A Ruby APNs HTTP/2 gem able to provide instant feedback"
    
* [autoprefixer-rails](https://github.com/ai/autoprefixer-rails) - "Autoprefixer is a tool to parse CSS and add vendor prefixes to CSS rules using values from the Can I Use database. This gem provides Ruby and Ruby on Rails integration with this JavaScript tool"
    
* [clockwork](https://github.com/Rykian/clockwork) - "Clockwork is a cron replacement. It runs as a lightweight, long-running Ruby process which sits alongside your web processes (Mongrel/Thin) and your worker processes (DJ/Resque/Minion/Stalker) to schedule recurring work at particular times or dates"
    
* [down](https://github.com/janko/down) - "Streaming downloads using net/http, http.rb, HTTPX or wget"
    
* [phlex-rails](https://github.com/phlex-ruby/phlex-rails) - "Phlex is a framework that lets you compose web views in pure Ruby"
    
* [premailer-rails](https://github.com/fphilipe/premailer-rails) - "This gem is a drop in solution for styling HTML emails with CSS without having to do the hard work yourself"
    
* [raindrops](https://rubygems.org/gems/raindrops) - "raindrops is a real-time stats toolkit to show statistics for Rack HTTP servers. It is designed for preforking servers such as unicorn, but should support any Rack HTTP server on platforms supporting POSIX shared memory"
    
* [strong\_migrations](https://github.com/ankane/strong_migrations) - "Catch unsafe migrations in development"
    
* [web-push](https://github.com/pushpad/web-push) - "This gem makes it possible to send push messages to web browsers from Ruby backends using the Web Push Protocol"
    
* [stripe-ruby-mock](https://github.com/stripe-ruby-mock/stripe-ruby-mock) - "A drop-in library to test stripe without hitting their servers"
    
* [rails-controller-testing](https://github.com/rails/rails-controller-testing) - "Brings back `assigns` and `assert_template` to your Rails tests"
    

There are many other gems used, I only selected few here. Browse the [Gemfile](https://github.com/feedbin/feedbin/blob/main/Gemfile) to discover more.

What could be mentioned is that they use their fork for some of the gems included in the file:

```ruby
# https://github.com/feedbin/feedbin/blob/main/Gemfile

# other gems

gem "rails", github: "feedbin/rails", branch: "7-1-stable-invalid-cache-entries"

# some other gems

gem "http",                github: "feedbin/http",                branch: "feedbin"
gem "carrierwave",         github: "feedbin/carrierwave",         branch: "feedbin"
gem "sax-machine",         github: "feedbin/sax-machine",         branch: "feedbin"
gem "feedjira",            github: "feedbin/feedjira",            branch: "f2"
gem "feedkit",             github: "feedbin/feedkit",             branch: "master"
gem "html-pipeline",       github: "feedbin/html-pipeline",       branch: "feedbin"
gem "html_diff",           github: "feedbin/html_diff",           ref: "013e1bb"
gem "twitter",             github: "feedbin/twitter",             branch: "feedbin"

# other gems 

group :development, :test do
  gem "stripe-ruby-mock", github: "feedbin/stripe-ruby-mock", branch: "feedbin", require: "stripe_mock"
# other gems
end

# other gem groups
```

### Code & Design Patterns

#### Code Organisation

Under `/app` there are 3 folders different from the ones that Rails comes with:

* `presenters`
    
* `uploaders`
    
* `validators`
    

The `lib` folder includes very few extra objects. Most of them seems to be related to communicating with external services.

Maybe worth mentioning from `lib` folder is the [`ConditionalSassCompressor`](https://github.com/feedbin/feedbin/blob/main/lib/conditional_sass_compressor.rb#L1)

```ruby
# https://github.com/feedbin/feedbin/blob/main/lib/conditional_sass_compressor.rb#L1

class ConditionalSassCompressor
  def compress(string)
    return string if string =~ /tailwindcss/
    options = { syntax: :scss, cache: false, read_cache: false, style: :compressed}
    begin
      Sprockets::Autoload::SassC::Engine.new(string, options).render
    rescue => e
      puts "Could not compress '#{string[0..65]}'...: #{e.message}, skipping compression"
      string
    end
  end
end
```

This is used to configure:

```ruby
# https://github.com/feedbin/feedbin/blob/main/config/application.rb#L47
config.assets.css_compressor = ConditionalSassCompressor.new
```

#### Routes

There is a combination of RESTful routes and non-restful routes.

Here is an example from `entries` in the [`routes.rb`](https://github.com/feedbin/feedbin/blob/main/config/routes.rb#L133) :

```ruby
# https://github.com/feedbin/feedbin/blob/main/config/routes.rb#L133

  resources :entries, only: [:show, :index, :destroy] do
    member do
      post :content
      post :unread_entries, to: "unread_entries#update"
      post :starred_entries, to: "starred_entries#update"
      post :mark_as_read, to: "entries#mark_as_read"
      post :recently_read, to: "recently_read_entries#create"
      post :recently_played, to: "recently_played_entries#create"
      get :push_view
      get :newsletter
    end
    collection do
      get :starred
      get :unread
      get :preload
      get :search
      get :recently_read, to: "recently_read_entries#index"
      get :recently_played, to: "recently_played_entries#index"
      get :updated, to: "updated_entries#index"
      post :mark_all_as_read
      post :mark_direction_as_read
    end
  end
```

#### Controllers

The controllers are mostly what I would call *vanilla Rails controllers*.

Three notes about them:

* Some of them are responding with JS usually using USJ or JQuery to change elements from the page.
    
* They contain non-Rails standard actions (actions that are not `show`, `index`, `new`, `create` ...)
    
* There is a namespaced `api` folder that contains APIs used by mobile apps
    

Here is one simple example for `DELETE /entries/:id` , the controller looks like this:

```ruby
# https://github.com/feedbin/feedbin/blob/main/app/controllers/entries_controller.rb#L238
  def destroy
    @user = current_user
    @entry = @user.entries.find(params[:id])
    if @entry.feed.pages?
      EntryDeleter.new.delete_entries(@entry.feed_id, @entry.id)
    end
  end
```

And here is the view [`destroy.js.erb`](https://github.com/feedbin/feedbin/blob/main/app/views/entries/destroy.js.erb#L1) :

```javascript
$('[data-behavior~=entries_target] [data-entry-id=<%= @entry.id %>]').remove();

feedbin.Counts.get().removeEntry(<%= @entry.id %>, <%= @entry.feed_id %>, 'unread')
feedbin.Counts.get().removeEntry(<%= @entry.id %>, <%= @entry.feed_id %>, 'starred')
feedbin.applyCounts(true)

feedbin.clearEntry();
feedbin.fullScreen(false)
```

The main pattern adopted to controllers is to have some logic in them and delegate to jobs some part of the processing.

The repo contains mostly straight-forward controllers like this one:

```ruby
# https://github.com/feedbin/feedbin/blob/main/app/controllers/pages_internal_controller.rb#L1
class PagesInternalController < ApplicationController

  def create
    @entry = SavePage.new.perform(current_user.id, params[:url], nil)
    get_feeds_list
  end
end
```

But also few controllers that include some logic:

```ruby
# https://github.com/feedbin/feedbin/blob/main/app/controllers/api/podcasts/v1/feeds_controller.rb#L8

 def show
  url = hex_decode(params[:id])
  @feed = Feed.find_by_feed_url(url)
  if @feed.present?
    if @feed.standalone_request_at.blank?
      FeedStatus.new.perform(@feed.id)
      FeedUpdate.new.perform(@feed.id)
    end
  else
    feeds = FeedFinder.feeds(url)
    @feed = feeds.first
  end

  if @feed.present?
    @feed.touch(:standalone_request_at)
  else
    status_not_found
  end
rescue => exception
  if Rails.env.production?
    ErrorService.notify(exception)
    status_not_found
  else
    raise exception
  end
end
```

Even with this structure, I find all controllers easy to read and I think they can be easier to change.

#### Models

The `app/models` folders contain both ActiveRecord and normal Ruby objects. With few exceptions, they are not namespaced.

#### Jobs

The `jobs` folder contains Sidekiq jobs which are used to do processing on various objects. They are usually called from controllers and most of them are async.

Here is one job that is caching views:

```ruby
# https://github.com/feedbin/feedbin/blob/main/app/jobs/cache_entry_views.rb#L1

class CacheEntryViews
  include Sidekiq::Worker
  include SidekiqHelper

  SET_NAME = "#{name}-ids"

  def perform(entry_id, process = false)
    if process
      cache_views
    else
      add_to_queue(SET_NAME, entry_id)
    end
  end

  def cache_views
    entry_ids = dequeue_ids(SET_NAME)
    entries = Entry.where(id: entry_ids).includes(feed: [:favicon])
    ApplicationController.render({
      partial: "entries/entry",
      collection: entries,
      format: :html,
      cached: true
    })
    ApplicationController.render({
      layout: nil,
      template: "api/v2/entries/index",
      assigns: {entries: entries},
      format: :html,
      locals: {
        params: {mode: "extended"}
      }
    })
  end
end
```

#### Presenters

There is a [`BasePresenter`](https://github.com/feedbin/feedbin/blob/main/app/presenters/base_presenter.rb#L1) and all other presenters are extending it via inheritance:

This controller defines a private method called `presents`:

```ruby
# https://github.com/feedbin/feedbin/blob/main/app/presenters/base_presenter.rb#L1

class BasePresenter
  def initialize(object, locals, template)
    @object = object
    @locals = locals
    @template = template
  end

  # ...

  private

  def self.presents(name)
    define_method(name) do
      @object
    end
  end
end
```

and it is used like this for example:

```ruby
# https://github.com/feedbin/feedbin/blob/main/app/presenters/user_presenter.rb#L2

class UserPresenter < BasePresenter
  presents :user
  delegate_missing_to :user

  # ... more code

  def theme
    result = settings["theme"].present? ? settings["theme"] : nil
    result || user.theme || "auto"
  end
  # ... other code
end
```

To use the presenters, there is a helper defined in `ApplicationHelper` will instantiate the proper helper based on the object class:

```ruby
module ApplicationHelper
  def present(object, locals = nil, klass = nil)
    klass ||= "#{object.class}Presenter".constantize
    presenter = klass.new(object, locals, self)
    yield presenter if block_given?
    presenter
  end
  
  # more code ...
end
```

and it is used [like this in views](https://github.com/feedbin/feedbin/blob/main/app/views/layouts/settings.html.erb#L1):

```erb
<% present @user do |user_presenter| %>
    <% @class = "settings-body settings-#{params[:action]} theme-#{user_presenter.theme}"%>
<% end %>
```

#### ApplicationComponents

Components are based on Phlex and they inherit from [`ApplicationComponent`](https://github.com/feedbin/feedbin/blob/main/app/views/components/application_component.rb#L3)

It defines a method to add Stimulus controller in components like this:

```ruby
# https://github.com/feedbin/feedbin/blob/main/app/views/components/application_component.rb#L25

  def stimulus(controller:, actions: {}, values: {}, outlets: {}, classes: {}, data: {})
    stimulus_controller = controller.to_s.dasherize

    action = actions.map do |event, function|
      "#{event}->#{stimulus_controller}##{function.camelize(:lower)}"
    end.join(" ").presence

    values.transform_keys! do |key|
      [controller, key, "value"].join("_").to_sym
    end

    outlets.transform_keys! do |key|
      [controller, key, "outlet"].join("_").to_sym
    end

    classes.transform_keys! do |key|
      [controller, key, "class"].join("_").to_sym
    end

    { controller: stimulus_controller, action: }.merge!({ **values, **outlets, **classes, **data})
  end
```

Where we can also see a bit of hash literal omission at `{controller: stimulus_controller, action: }`

But more interesting that this method that helps defining a Stimulus controller, is the method used to define a Stimulus item that uses binding to get variables from the object where it is used:

```ruby
# https://github.com/feedbin/feedbin/blob/main/app/views/components/application_component.rb#L47

  def stimulus_item(target: nil, actions: {}, params: {}, data: {}, for:)
    stimulus_controller = binding.local_variable_get(:for).to_s.dasherize

    action = actions.map do |event, function|
      "#{event}->#{stimulus_controller}##{function.to_s.camelize(:lower)}"
    end.join(" ").presence

    params.transform_keys! do |key|
      :"#{binding.local_variable_get(:for)}_#{key}_param"
    end

    defaults = { **params, **data }

    if action
      defaults[:action] = action
    end

    if target
      defaults[:"#{binding.local_variable_get(:for)}_target"] = target.to_s.camelize(:lower)
    end

    defaults
  end
```

The part with `binding` does the following:

* `stimulus_controller = binding.local_variable_get(:for).to_s.dasherize` This line retrieves the value of the local variable `for`, converts it to a string, and then applies the dasherize method (presumably to format it for use in a specific context, like a CSS class or an identifier in HTML).
    
* Apparently `binding.local_variable_get` should not be needed as the variable is passed a keyword parameter to the method. But the name of the variable is `for` which is a reserved word and thus if the code would have been `stimulus_controller = for.to_s_dasherize` that would have raised `syntax error, unexpected '.' (SyntaxError)`
    

This is a way to have keyword arguments named as reserved words and still be able to use them.

#### ComponentsPreview

All components can be previewed via Lookbook and they can be found in `test/components`

## Testing

For testing it uses Minitest, the default testing framework from Rails. It uses fixtures to set up the test db.

Tests are simple and direct, containing all preconditions and postconditions in each test. This is great for following what each test is doing.

There are controller tests, model tests, job tests and some system tests. There are more controller tests than system tests making the test suite run quite fast. Also the jobs are covered pretty good with testing as there is a log of logic in the jobs.

#### Custom assertions

There are some custom assertions created specifically to work with collections: `assert_has_keys` will check if all keys are included in the hash and `assert_equal_ids` will check if the two collections provided have the same ids (one being a collection of objects and the other one being a hash).

```ruby
# https://github.com/feedbin/feedbin/blob/main/test/support/assertions.rb#L3

    def assert_has_keys(keys, hash)
      assert(keys.all? { |key| hash.key?(key) })
    end

    def assert_equal_ids(collection, results)
      expected = Set.new(collection.map(&:id))
      actual = Set.new(results.map { |result| result["id"] })
      assert_equal(expected, actual)
    end
```

## Conclusion

In conclusion, Feedbin is an open-source project that combines RSS feeds and newsletters into a convenient platform.

It utilizes Ruby on Rails, PostgreSQL, Sidekiq, and various other technologies to provide a robust and efficient service.

The code is well-organized and simple to follow the logic and what is happening. I think it will make it easy for anyone to contribute to this repo. If you want to run this yourself locally you should take a look at the [feedbin-docker](https://github.com/angristan/feedbin-docker).

---

Enjoyed this article?

👉 Join my [**Short Ruby News**](https://shortruby.com/) **newsletter** for weekly Ruby updates from the community and visit [**rubyandrails.info**](http://rubyandrails.info)**,** a directory with learning content about Ruby.

👐 Subscribe to my Ruby and Ruby on rails courses over email at [learn.shortruby.com](https://learn.shortruby.com) - effortless learning anytime, anywhere

🤝 Let's connect on [**Ruby.social**](https://ruby.social/@lucian) or [**Linkedin**](https://linkedin.com/in/lucianghinda) or [**Twitter**](https://x.com/lucianghinda) where I post mainly about Ruby and Rails.

🎥 Follow me on [**my YouTube channel**](https://www.youtube.com/@shortruby) for short videos about Ruby