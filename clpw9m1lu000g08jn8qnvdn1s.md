---
title: "Ruby on Rails Open Source: Mastodon"
seoTitle: "Mastodon: Ruby on Rails Open Source Web App"
seoDescription: "Explore Mastodon, an open-source, federated social network prioritizing privacy and freedom of expression, built with Ruby, Rails, React, and Redux"
datePublished: Fri Dec 08 2023 06:47:47 GMT+0000 (Coordinated Universal Time)
cuid: clpw9m1lu000g08jn8qnvdn1s
slug: ruby-on-rails-open-source-mastodon
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1702017876116/cf6f3bd5-5c5b-4588-9add-af1ee3dd4cf0.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1702017946603/b6c24db8-6ab3-4785-b06f-c33c10acff54.png
tags: programming-blogs, ruby, opensource, ruby-on-rails, programming-languages

---

## The product

[https://joinmastodon.org](https://joinmastodon.org)

> Mastodon is a free, open-source social network server based on ActivityPub where users can follow friends and discover new ones. On Mastodon, users can publish anything they want: links, pictures, text, and video. All Mastodon servers are interoperable as a federated network.

[![a social networking site with a purple background](https://cdn.hashnode.com/res/hashnode/image/upload/v1701315312358/e58506b3-c3e8-49df-a36a-c0dd870a2a05.png align="center")](https://joinmastodon.org)

## Open source

The project is open source at [https://github.com/mastodon/mastodon](https://github.com/mastodon/mastodon)

### License

The license is [GNU Affero General Public License v3.0](https://github.com/mastodon/mastodon/blob/main/LICENSE)

[![A screenshot of the https://github.com/mastodon/mastodon/blob/main/LICENSE](https://cdn.hashnode.com/res/hashnode/image/upload/v1701315413022/8107298f-e08b-4f0d-bf98-55b3e9675d7b.png align="center")](https://github.com/mastodon/mastodon/blob/main/LICENSE)

## Technical review

### Architecture

**Ruby on Rails** - used for API and some of the web pages

**React and Redux** - used for dynamic parts of the interface

**Node.js** - used for streaming API

I will focus on this review on the Ruby on Rails part.

### Ruby and Rails version

It uses **Ruby 3.2.2** and (at the moment of writing this review) **Rails 7.1.1**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701315843968/0edcf5c5-72a8-43a9-bc84-c5d07647f1eb.png align="center")

### Stats

Running `bin/rails stats` will give the following:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701315952061/480be8b9-bcc7-49f7-9198-bc01ff14ad8e.png align="center")

Running the VScodeCounter will give the following stats:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701316044455/ed368ba6-ca1d-4823-9bac-0cba858e2298.png align="center")

There are very few actual XML files. So what is marked there as XML is probably a flavour of HTML either from the Ruby on Rails side or from the React/Nodejs side.

## Style Guide

They are using Rubocop to enforce coding guidelines. They are using along with standard Rubocop:

* [**rubocop-rails**](https://github.com/rubocop/rubocop-rails/?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Automatic Rails code style checking tool. A RuboCop extension focused on enforcing Rails best practices and coding conventions."*
    
* [**rubocop-performance**](https://github.com/rubocop/rubocop-performance/?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"A collection of RuboCop cops to check for performance optimizations in Ruby code."*
    
* [**rubocop-capybara**](https://github.com/rubocop/rubocop-capybara?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *" Code style checking for Capybara test files (RSpec, Cucumber, Minitest). A plugin for the RuboCop code style enforcing & linting tool"*
    
* [**rubocop-rspec**](https://github.com/rubocop/rubocop-rspec?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *" Code style checking for RSpec files. A plugin for the RuboCop code style enforcing & linting tool"*
    

And a custom cop called [MiddleDot](https://github.com/mastodon/mastodon/blob/main/lib/linter/rubocop_middle_dot.rb#L3) that does the following:

> Bans the usage of “•” (bullet) in HTML/HAML in favour of “·” (middle dot) in string literals

Here are some extra configurations for Rubocop:

* `DisplayStyleGuide: true` so that it will display the URL for the Style guide when there is an offense.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701317172965/8062b808-74d6-4d91-8a19-598ced0b9fa6.png align="center")

* `UseCache: true` - to store and reuse results for making the Rubocop run faster
    

What I noticed and liked is that for each Rubocop rule where they overwrite, they provide as a comment the URL for that rule. This makes it easier to navigate to it and read about it.

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701317390401/614f3bae-e28e-4d4d-bdfb-5f04d162e0b2.png align="center")](https://github.com/mastodon/mastodon/blob/main/.rubocop.yml#L34)

## Storage Persistence and in-memory storage

The database used is PostgreSQL and they configure by default a primary and replica DB:

The Database pool is set by using the following:

```yaml
# https://github.com/mastodon/mastodon/blob/main/config/database.yml#L3
pool: <%= ENV["DB_POOL"] || (if Sidekiq.server? then Sidekiq[:concurrency] else ENV['MAX_THREADS'] end) || 5 %>
```

So if there is not `DB_POOL` environment it will try to use `Sidekiq[:concurrency]` else it will default to `` `MAX_THREADS` ``

As in-memory storage it uses Redis and there is a `RedisConfiguration` object that helps with configuring the connection pool:

```ruby
# https://github.com/mastodon/mastodon/blob/main/app/lib/redis_configuration.rb#L3

class RedisConfiguration
  class << self
    def establish_pool(new_pool_size)
      @pool&.shutdown(&:close)
      @pool = ConnectionPool.new(size: new_pool_size) { new.connection }
    end

    delegate :with, to: :pool

    def pool
      @pool ||= establish_pool(pool_size)
    end

    def pool_size
      if Sidekiq.server?
        Sidekiq[:concurrency]
      else
        ENV['MAX_THREADS'] || 5
      end
    end
  end

  def connection
    if namespace?
      Redis::Namespace.new(namespace, redis: raw_connection)
    else
      raw_connection
    end
  end
# ... other method
end
```

It uses a class instance variables `@pool` to share the connection pool across multiple instances of this class and the size is by default `Sidekiq[:concurrency]` size if Sidekiq used or `MAX_THREADS` value.

If there is an environment variable `REDIS_NAMESPACE` it will use Redis under that specific namespace.

Further down this `RedisConfiguration` is added to other objects (controllers for example) by using a module called [Redisable](https://github.com/mastodon/mastodon/blob/main/app/models/concerns/redisable.rb#L3)

```ruby
# https://github.com/mastodon/mastodon/blob/main/app/models/concerns/redisable.rb#L3

module Redisable
  def redis
    Thread.current[:redis] ||= RedisConfiguration.pool.checkout
  end

  def with_redis(&block)
    RedisConfiguration.with(&block)
  end
end
```

There is then a module called [Lockable](https://github.com/mastodon/mastodon/blob/main/app/models/concerns/lockable.rb#L3) that defines `with_redis_lock`:

```ruby
# https://github.com/mastodon/mastodon/blob/main/app/models/concerns/lockable.rb#L3

module Lockable
  # @param [String] lock_name
  # @param [ActiveSupport::Duration] autorelease Automatically release the lock after this time
  # @param [Boolean] raise_on_failure Raise an error if a lock cannot be acquired, or fail silently
  # @raise [Mastodon::RaceConditionError]
  def with_redis_lock(lock_name, autorelease: 15.minutes, raise_on_failure: true)
    with_redis do |redis|
      RedisLock.acquire(redis: redis, key: "lock:#{lock_name}", autorelease: autorelease.seconds) do |lock|
        if lock.acquired?
          yield
        elsif raise_on_failure
          raise Mastodon::RaceConditionError, "Could not acquire lock for #{lock_name}, try again later"
        end
      end
    end
  end
end
```

They are used for example in a [controller](https://github.com/mastodon/mastodon/blob/main/app/controllers/api/v1/statuses/reblogs_controller.rb#L3):

```ruby
# https://github.com/mastodon/mastodon/blob/main/app/controllers/api/v1/statuses/reblogs_controller.rb#L3

class Api::V1::Statuses::ReblogsController < Api::V1::Statuses::BaseController
  include Redisable
  include Lockable

  # ... callbacks

  def create
    with_redis_lock("reblog:#{current_account.id}:#{@reblog.id}") do
      @status = ReblogService.new.call(current_account, @reblog, reblog_params)
    end

    render json: @status, serializer: REST::StatusSerializer
  end
  # ... other methods
end
```

## Gems used

I picked some gems from the [Gemfile](https://github.com/mastodon/mastodon/blob/main/Gemfile#L3) that I found interesting to mention:

* [addressable](https://github.com/sporkmonger/addressable?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - "Addressable is an alternative implementation to the URI implementation that is part of Ruby's standard library. It is flexible, offers heuristic parsing, and additionally provides extensive support for IRIs and URI templates"
    
* [**blurhash**](https://github.com/Gargron/blurhash?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Encode an image as a small string that can saved in the database and used to show a blurred preview before the real image loads"*
    
* [**charlock\_holmes**](https://github.com/brianmario/charlock_holmes?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"charlock\_holmes provides binary and text detection as well as text transcoding using libicu"*
    
* [**chewy**](https://github.com/toptal/chewy?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Chewy provides functionality for Elasticsearch index handling, documents import mappings and chainable query DSL"*
    
* [**climate\_control**](https://github.com/thoughtbot/climate_control?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Modify your ENV"*
    
* [**cocoon**](http://github.com/nathanvda/cocoon?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Unobtrusive nested forms handling, using jQuery. Use this and discover cocoon-heaven."*
    
* [**color\_diff**](https://github.com/hansondr/color_diff?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Calculate RGB color distances using CIEDE2000 formula"*
    
* [**discard**](https://github.com/jhawthorn/discard?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Allows marking ActiveRecord objects as discarded, and provides scopes for filtering."*
    
* [**doorkeeper**](https://github.com/doorkeeper-gem/doorkeeper?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Doorkeeper is an OAuth 2 provider for Rails and Grape."*
    
* [**ed25519**](https://github.com/RubyCrypto/ed25519?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"A Ruby binding to the Ed25519 elliptic curve public-key signature system described in RFC 8032."*
    
* [**email\_spec**](http://github.com/email-spec/email-spec/?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Easily test email in RSpec, Cucumber, and MiniTest"*
    
* [**fabrication**](https://gitlab.com/fabrication-gem/fabrication?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Fabrication is an object generation framework for ActiveRecord, Mongoid, DataMapper, Sequel, or any other Ruby object."*
    
* [**fast\_blank**](https://github.com/SamSaffron/fast_blank?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Provides a C-optimized method for determining if a string is blank"*
    
* [**fastimage**](http://github.com/sdsykes/fastimage?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"FastImage finds the size or type of an image given its uri by fetching as little as needed."*
    
* [**fuubar**](https://github.com/thekompanee/fuubar?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"the instafailing RSpec progress bar formatter"*
    
* [**haml\_lint**](https://github.com/sds/haml-lint?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Configurable tool for writing clean and consistent HAML"*
    
* [**hcaptcha**](https://github.com/Nexus-Mods/hcaptcha?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Ruby helpers for hCaptcha"*
    
* [**htmlentities**](https://github.com/threedaymonk/htmlentities?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"A module for encoding and decoding (X)HTML entities."*
    
* [**http\_accept\_language**](https://github.com/iain/http_accept_language?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Find out which locale the user preferes by reading the languages they specified in their browser"*
    
* [**httplog**](https://github.com/trusche/httplog?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Log outgoing HTTP requests made from your application. Helpful for tracking API calls of third party gems that don't provide their own log output."*
    
* [**i18n-tasks**](https://github.com/glebm/i18n-tasks?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"i18n-tasks helps you find and manage missing and unused translations. It analyses code statically for key usages, such as* `I18n.t('some.key')`*, in order to report keys that are missing or unused, pre-fill missing keys (optionally from Google Translate), and remove unused keys. "*
    
* [**idn-ruby**](http://github.com/deepfryed/idn-ruby?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *" Ruby Bindings for the GNU LibIDN library, an implementation of the Stringprep, Punycode and IDNA specifications defined by the IETF Internationalized Domain Names (IDN) working group. Included are the most important parts of the Stringprep, Punycode and IDNA APIs like performing Stringprep processings, encoding to and decoding from Punycode strings and converting entire domain names to and from the ACE encoded form. "*
    
* [**json-ld**](https://github.com/ruby-rdf/json-ld?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"JSON::LD parses and serializes JSON-LD into RDF and implements expansion, compaction and framing API interfaces for the Ruby RDF.rb library suite."*
    
* [**json-ld-preloaded**](https://github.com/ruby-rdf/json-ld-preloaded?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"A meta-release of the json-ld gem including preloaded vocabularies for the Ruby RDF.rb library suite."*
    
* [**json-schema**](http://github.com/voxpupuli/json-schema/?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Ruby JSON Schema Validator"*
    
* [**kt-paperclip**](https://github.com/kreeti/kt-paperclip?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Easy upload management for ActiveRecord"*
    
* [**link\_header**](https://github.com/asplake/link_header?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Converts conforming link headers to and from text, LinkHeader objects and corresponding (JSON-friendly) Array representations, also HTML link elements."*
    
* [**mario-redis-lock**](https://github.com/marioizquierdo/mario-redis-lock?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Yet another Ruby distributed lock using Redis, with emphasis in transparency. Requires Redis &gt;= 2.6.12, because it uses the new syntax for SET to easily implement the robust algorithm described in the SET command documentation (*[*http://redis.io/commands/set*](https://github.com/marioizquierdo/mario-redis-lock?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com)*)."*
    
* [**memory\_profiler**](https://github.com/SamSaffron/memory_profiler?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Memory profiling routines for Ruby 2.5+"*
    
* [**nsa**](https://github.com/localshred/nsa?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Listen to your Rails ActiveSupport::Notifications and deliver to a Statsd compatible backend"*
    
* [**oj**](https://github.com/ohler55/oj?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"The fastest JSON parser and object serializer."*
    
* [**ox**](https://github.com/ohler55/ox?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"A fast XML parser and object serializer that uses only standard C lib. Optimized XML (Ox), as the name implies was written to provide speed optimized XML handling. It was designed to be an alternative to Nokogiri and other Ruby XML parsers for generic XML parsing and as an alternative to Marshal for Object serialization. "*
    
* [**parslet**](https://github.com/kschiess/parslet?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Parser construction library with great error reporting in Ruby."*
    
* [**pghero**](https://github.com/ankane/pghero?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"A performance dashboard for Postgres"*
    
* [**private\_address\_check**](https://github.com/jtdowney/private_address_check?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Checks if a IP or hostname would cause a request to a private network (RFC 1918)"*
    
* [**public\_suffix**](https://github.com/weppos/publicsuffix-ruby/tree/v5.0.4?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"PublicSuffix can parse and decompose a domain name into top level domain, domain and subdomains."*
    
* [**redis-namespace**](https://github.com/resque/redis-namespace?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Adds a Redis::Namespace class which can be used to namespace calls to Redis. This is useful when using a single instance of Redis with multiple, different applications. "*
    
* [**ruby-progressbar**](https://github.com/jfelchner/ruby-progressbar?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Ruby/ProgressBar is an extremely flexible text progress bar library for Ruby. The output can be customized with a flexible formatting system including: percentage, bars of various formats, elapsed time and estimated time remaining."*
    
* [**scenic**](https://github.com/scenic-views/scenic?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *" Adds methods to ActiveRecord::Migration to create and manage database views in Rails "*
    
* [**sidekiq-bulk**](https://github.com/aprescott/sidekiq-bulk?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Augments Sidekiq job classes with a push\_bulk method for easier bulk pushing."*
    
* [**sidekiq-unique-jobs**](https://github.com/mhenrixon/sidekiq-unique-jobs?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Prevents simultaneous Sidekiq jobs with the same unique arguments to run. Highly configurable to suite your specific needs. "*
    
* [**simple-navigation**](http://github.com/codeplant/simple-navigation?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"With the simple-navigation gem installed you can easily create multilevel navigations for your Rails, Sinatra or Padrino applications. The navigation is defined in a single configuration file. It supports automatic as well as explicit highlighting of the currently active navigation through regular expressions."*
    
* [**stoplight**](https://github.com/orgsync/stoplight?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"An implementation of the circuit breaker pattern."*
    
* [**tty-prompt**](https://github.com/piotrmurach/tty-prompt?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"A beautiful and powerful interactive command line prompt with a robust API for getting and validating complex inputs."*
    
* [**twitter-text**](https://github.com/twitter/twitter-text?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"A gem that provides text handling for Twitter"*
    
* [**xorcist**](https://github.com/fny/xorcist?utm_source=shortruby&utm_campaign=shortruby_0066&ref=shortruby.com) - *"Blazing-fast-cross-platform-monkey-patch-free string XOR. Yes, that means JRuby too."*
    

## Code & Design Patterns

### Folders

Here are some non-standard Rails folders:

* `app/chewy` - containing Chewy objects
    
* `app/policies` - containing objects that implement policies used by Pundit gem
    
* `app/presenters` - contain objects that implement `ActiveModelSerializers::Model` interface or that exposes methods that return hashes
    
* `app/services` - contain objects that implement Service Object pattern
    
* `app/validators` - contain objects that mostly implement `ActiveModel::Validator` interface
    
* `app/workers` - contain objects that implement `Sidekiq::Worker` interface
    
* `app/lib` - contains probably POROs that do not belong anywhere else
    
* `dist` - contains [systemd](https://systemd.io) files used to manage all web app services on Linux-based systems with [systemd](https://systemd.io) installed
    
* `lib` - contains code that either extends or changes other libraries objects or that extends external libraries used
    

### Controllers

There are 2 main categories of controllers when thinking about how the information will be displayed:

* General controllers (like the ones that are directly under `app/controllers` or `app/controllers/admin`) that render an SSR view that will write some HTML tags and then load the React app. They respond to `html` and `json`
    
* The controllers under `app/api` that only responds to `json`
    

#### Caching

It implements the [`Vary`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Vary) header:

> The Vary HTTP response header describes the parts of the request message aside from the method and URL that influenced the content of the response it occurs in. Most often, this is used to create a cache key when content negotiation is in use.

```ruby
# https://github.com/mastodon/mastodon/blob/main/app/controllers/concerns/cache_concern.rb#L150
def vary_by(value, **kwargs)
  before_action(**kwargs) do |controller|
    response.headers['Vary'] = value.respond_to?(:call) ? controller.instance_exec(&value) : value
  end
end

# Used like this

# https://github.com/mastodon/mastodon/blob/main/app/controllers/follower_accounts_controller.rb#L3
vary_by -> { public_fetch_mode? ? 'Accept, Accept-Language, Cookie' : 'Accept, Accept-Language, Cookie, Signature' }
```

A note about using `instance_exec` is a method from [BasicObject#instance\_eval](https://docs.ruby-lang.org/en/3.2/BasicObject.html#method-i-instance_exec) that can be used to execute a block within the context of the receiver.

Another method used is [`render_with_cache`](https://github.com/mastodon/mastodon/blob/main/app/controllers/concerns/cache_concern.rb#L172) used by API controllers to cache JSON responses:

```ruby
# https://github.com/mastodon/mastodon/blob/main/app/controllers/concerns/cache_concern.rb#L172 

def render_with_cache(**options)
  raise ArgumentError, 'Only JSON render calls are supported' unless options.key?(:json) || block_given?

  key        = options.delete(:key) || [[params[:controller], params[:action]].join('/'), options[:json].respond_to?(:cache_key) ? options[:json].cache_key : nil, options[:fields].nil? ? nil : options[:fields].join(',')].compact.join(':')
  expires_in = options.delete(:expires_in) || 3.minutes
  body       = Rails.cache.read(key, raw: true)

  if body
    render(options.except(:json, :serializer, :each_serializer, :adapter, :fields).merge(json: body))
  else
    if block_given?
      options[:json] = yield
    elsif options[:json].is_a?(Symbol)
      options[:json] = send(options[:json])
    end

    render(options)
    Rails.cache.write(key, response.body, expires_in: expires_in, raw: true)
  end
end

# Used like this:
# https://github.com/mastodon/mastodon/blob/main/app/controllers/accounts_controller.rb#L35

format.json do
  expires_in 3.minutes, public: !(authorized_fetch_mode? && signed_request_account.present?)
  render_with_cache json: @account, content_type: 'application/activity+json', serializer: ActivityPub::ActorSerializer, adapter: ActivityPub::Adapter
end
```

And a third method that is used is [`cache_collection`](https://github.com/mastodon/mastodon/blob/main/app/controllers/concerns/cache_concern.rb#L193-L194) used with models that include the [`Cacheable`](https://github.com/mastodon/mastodon/blob/main/app/models/concerns/cacheable.rb#L3) concern:

```ruby
# https://github.com/mastodon/mastodon/blob/main/app/controllers/concerns/cache_concern.rb#L193-L194

def cache_collection(raw, klass)
  return raw unless klass.respond_to?(:with_includes)

  raw = raw.cache_ids.to_a if raw.is_a?(ActiveRecord::Relation)
  return [] if raw.empty?

  cached_keys_with_value = begin
    Rails.cache.read_multi(*raw).transform_keys(&:id).transform_values { |r| ActiveRecordCoder.load(r) }
  rescue ActiveRecordCoder::Error
    {} # The serialization format may have changed, let's pretend it's a cache miss.
  end

  uncached_ids = raw.map(&:id) - cached_keys_with_value.keys

  klass.reload_stale_associations!(cached_keys_with_value.values) if klass.respond_to?(:reload_stale_associations!)

  unless uncached_ids.empty?
    uncached = klass.where(id: uncached_ids).with_includes.index_by(&:id)

    uncached.each_value do |item|
      Rails.cache.write(item, ActiveRecordCoder.dump(item))
    end
  end

  raw.filter_map { |item| cached_keys_with_value[item.id] || uncached[item.id] }
end

# Used like this:

format.rss do
  expires_in 1.minute, public: true

  limit     = params[:limit].present? ? [params[:limit].to_i, PAGE_SIZE_MAX].min : PAGE_SIZE
  @statuses = filtered_statuses.without_reblogs.limit(limit)
  @statuses = cache_collection(@statuses, Status)
end
```

#### Controller with multiple respond\_to

Here is an example of a controller action with multiple respond\_to formats:

```ruby
# https://github.com/mastodon/mastodon/blob/main/app/controllers/accounts_controller.rb#L17
def show
  respond_to do |format|
    format.html do
      expires_in(15.seconds, public: true, stale_while_revalidate: 30.seconds, stale_if_error: 1.hour) unless user_signed_in?

      @rss_url = rss_url
    end

    format.rss do
      expires_in 1.minute, public: true

      limit     = params[:limit].present? ? [params[:limit].to_i, PAGE_SIZE_MAX].min : PAGE_SIZE
      @statuses = filtered_statuses.without_reblogs.limit(limit)
      @statuses = cache_collection(@statuses, Status)
    end

    format.json do
      expires_in 3.minutes, public: !(authorized_fetch_mode? && signed_request_account.present?)
      render_with_cache json: @account, content_type: 'application/activity+json', serializer: ActivityPub::ActorSerializer, adapter: ActivityPub::Adapter
    end
  end
end
```

#### ApplicationController

It includes a list of concerns, defines some helper methods and define a couple of `rescue_from` responses.

What I discovered here and I find useful to have a common language when writing custom controllers:

`truthy_param?` the method that I think is useful to use to check if a param is a truthy value:

```ruby
# https://github.com/mastodon/mastodon/blob/main/app/controllers/application_controller.rb#L91
def truthy_param?(key)
  ActiveModel::Type::Boolean.new.cast(params[key])
end

# Used like this

# https://github.com/mastodon/mastodon/blob/main/app/controllers/api/v1/timelines/public_controller.rb#L37-L44
def public_feed
  PublicFeed.new(
    current_account,
    local: truthy_param?(:local),
    remote: truthy_param?(:remote),
    only_media: truthy_param?(:only_media)
  )
end
```

A series of methods that can be used to respond with errors:

```ruby
# https://github.com/mastodon/mastodon/blob/main/app/controllers/application_controller.rb#L169-L174

# First there is a respond_with_error defined: 
def respond_with_error(code)
  respond_to do |format|
    format.any  { render "errors/#{code}", layout: 'error', status: code, formats: [:html] }
    format.json { render json: { error: Rack::Utils::HTTP_STATUS_CODES[code] }, status: code }
  end
end

# Then there a series of methods defined like this:

# https://github.com/mastodon/mastodon/blob/main/app/controllers/application_controller.rb#L96
def forbidden
  respond_with_error(403)
end

def not_found
  respond_with_error(404)
end

def gone
  respond_with_error(410)
end
```

### Models

Some stats:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701852187787/445496b1-0088-4655-9190-d55a94670bef.png align="center")

* `189` files in `app/models` and `29` of them are in `app/models/concerns`
    
* `71` files are not directly inherited from `ApplicationRecord`
    
* `89` are `ActiveRecord` models
    

The `ApplicationRecord` model is connected to writing and reading databases and defines one common instance method `boolean_with_default` that will be used to define some getters that return a default value when nil. It is a simple method but very useful.

```ruby
# https://github.com/mastodon/mastodon/blob/main/app/models/application_record.rb#L16

def boolean_with_default(key, default_value)
  value = attributes[key]

  if value.nil?
    default_value
  else
    value
  end
end

# Used like this

# https://github.com/mastodon/mastodon/blob/main/app/models/tag.rb#L71

def usable
  boolean_with_default('usable', true)
end

alias usable? usable
```

Also, notice the choice to use `alias` to define the predicate method `usable?` instead of defining it as a normal method (e.g. `def usable? = usable`)

Here are some things that I found in the models:

#### **Counting cache**

The `Account` model has a couple of associated stats models implemented by using a `has_one :account_stat` that includes some colums like: `statuses_count`, `following_count`, `followers_count` . The count is incremented by executing the following SQL:

```ruby
# https://github.com/mastodon/mastodon/blob/main/app/models/concerns/account_counters.rb#L34

def update_count!(key, value)
# omitted code ...

  sql = if value.positive? && key == :statuses_count
          <<-SQL.squish
            INSERT INTO account_stats(account_id, #{key}, created_at, updated_at, last_status_at)
              VALUES (:account_id, :default_value, now(), now(), now())
            ON CONFLICT (account_id) DO UPDATE
            SET #{key} = account_stats.#{key} + :value,
                last_status_at = now(),
                updated_at = now()
            RETURNING id;
          SQL
        else
          <<-SQL.squish
            INSERT INTO account_stats(account_id, #{key}, created_at, updated_at)
              VALUES (:account_id, :default_value, now(), now())
            ON CONFLICT (account_id) DO UPDATE
            SET #{key} = account_stats.#{key} + :value,
                updated_at = now()
            RETURNING id;
          SQL
        end

  sql = AccountStat.sanitize_sql([sql, account_id: id, default_value: default_value, value: value])
  account_stat_id = AccountStat.connection.exec_query(sql)[0]['id']
  # Omitted code ..
end

# Used like this

def increment_count!(key) = update_count!(key, 1)
def decrement_count!(key) = update_count!(key, -1)
```

The reason for using the custom SQL is [explained](https://github.com/mastodon/mastodon/blob/main/app/models/concerns/account_counters.rb#L41-L44) in the same method:

> We do an upsert using manually written SQL, as Rails' upsert method does not seem to support writing expressions in the UPDATE clause, but only re-insert the provided values instead. Even ARel seem to be missing proper handling of upserts.

This will be used in the following way:

```ruby
class Favourite < ApplicationRecord
  after_create :increment_cache_counters
  belongs_to :status,  inverse_of: :favourites
  # Code omitted ... 
  private

  def increment_cache_counters
    status&.increment_count!(:favourites_count)
  end
  # code omitted ...
end
```

#### **User Roles**

User Roles are defined in the UserRole AR model using the bitwise shift left operator:

```ruby
class UserRole < ApplicationRecord
  FLAGS = {
    administrator: (1 << 0),
    view_devops: (1 << 1),
    view_audit_log: (1 << 2),
    view_dashboard: (1 << 3),
    manage_reports: (1 << 4),
    manage_federation: (1 << 5),
    manage_settings: (1 << 6),
    manage_blocks: (1 << 7),
    manage_taxonomies: (1 << 8),
    manage_appeals: (1 << 9),
    manage_users: (1 << 10),
    manage_invites: (1 << 11),
    manage_rules: (1 << 12),
    manage_announcements: (1 << 13),
    manage_custom_emojis: (1 << 14),
    manage_webhooks: (1 << 15),
    invite_users: (1 << 16),
    manage_roles: (1 << 17),
    manage_user_access: (1 << 18),
    delete_user_data: (1 << 19),
  }.freeze

  module Flags
    NONE = 0
    ALL  = FLAGS.values.reduce(&:|)

    DEFAULT = FLAGS[:invite_users]
    # code omitted ... 
  end

  # code omitted ... 
end
```

In case you want to understand how they are used, here are some expressions:

* If you execute `(1 << 2).to_s(2)` =&gt; `100`
    
* If you execute `(1 << 3).to_s(2)` =&gt; `1000`
    
* If you execute `((1<<2) | (1<<3)).to_s(2)` =&gt; `1100`
    
* If you execute `ALL.to_s(2)` will get `11111111111111111111`
    

Then in the UserRole model there are the following methods defined:

```ruby
# https://github.com/mastodon/mastodon/blob/main/app/models/user_role.rb#L164

def can?(*any_of_privileges)
  any_of_privileges.any? { |privilege| in_permissions?(privilege) }
end

def in_permissions?(privilege)
  raise ArgumentError, "Unknown privilege: #{privilege}" unless FLAGS.key?(privilege)

  computed_permissions & FLAGS[privilege] == FLAGS[privilege]
end
```

They are used in `AccountPolicy` like this:

```ruby
# https://github.com/mastodon/mastodon/blob/main/app/policies/account_policy.rb#L3

class AccountPolicy < ApplicationPolicy
  def index?
    role.can?(:manage_users)
  end

  def show?
    role.can?(:manage_users)
  end
# code omitted ...
end
```

### Services

#### BaseService

There is a simple `BaseService` that looks like this:

```ruby
# https://github.com/mastodon/mastodon/blob/main/app/services/base_service.rb#L3
class BaseService
  include ActionView::Helpers::TextHelper
  include ActionView::Helpers::SanitizeHelper

  include RoutingHelper

  def call(*)
    raise NotImplementedError
  end
end
```

and then almost all services are inherited from this `BaseService`

The services defined here can be called from controllers or from models.

Here is an example of the Follow service (will just add here the #call method):

* It documents the call using the YARD format
    
* It uses comments to explain why the home feed is marked as partial! (that will set a key in Redis to regenerate the home feed)
    

```ruby
# https://github.com/mastodon/mastodon/blob/main/app/services/follow_service.rb#L3

# Follow a remote user, notify remote user about the follow
# @param [Account] source_account From which to follow
# @param [Account] target_account Account to follow
# @param [Hash] options
# @option [Boolean] :reblogs Whether or not to show reblogs, defaults to true
# @option [Boolean] :notify Whether to create notifications about new posts, defaults to false
# @option [Array<String>] :languages Which languages to allow on the home feed from this account, defaults to all
# @option [Boolean] :bypass_locked
# @option [Boolean] :bypass_limit Allow following past the total follow number
# @option [Boolean] :with_rate_limit
def call(source_account, target_account, options = {})
  @source_account = source_account
  @target_account = target_account
  @options        = { bypass_locked: false, bypass_limit: false, with_rate_limit: false }.merge(options)

  raise ActiveRecord::RecordNotFound if following_not_possible?
  raise Mastodon::NotPermittedError  if following_not_allowed?

  if @source_account.following?(@target_account)
    return change_follow_options!
  elsif @source_account.requested?(@target_account)
    return change_follow_request_options!
  end

  ActivityTracker.increment('activity:interactions')

  # When an account follows someone for the first time, avoid showing
  # an empty home feed while the follow request is being processed
  # and the feeds are being merged
  mark_home_feed_as_partial! if @source_account.not_following_anyone?

  if (@target_account.locked? && !@options[:bypass_locked]) || @source_account.silenced? || @target_account.activitypub?
    request_follow!
  elsif @target_account.local?
    direct_follow!
  end
end
```

#### FetchLinkCardService

Here is a piece of code from [`FetchLinkCardService`](https://github.com/mastodon/mastodon/blob/main/app/services/fetch_link_card_service.rb#L7) that parses a status and extracts shared URLs.

First, it defines a Regexp for searching for URL patterns:

```ruby
# https://github.com/mastodon/mastodon/blob/main/app/services/fetch_link_card_service.rb#L7
  URL_PATTERN = %r{
    (#{Twitter::TwitterText::Regex[:valid_url_preceding_chars]})                                                                #   $1 preceding chars
    (                                                                                                                           #   $2 URL
      (https?://)                                                                                                               #   $3 Protocol (required)
      (#{Twitter::TwitterText::Regex[:valid_domain]})                                                                           #   $4 Domain(s)
      (?::(#{Twitter::TwitterText::Regex[:valid_port_number]}))?                                                                #   $5 Port number (optional)
      (/#{Twitter::TwitterText::Regex[:valid_url_path]}*)?                                                                      #   $6 URL Path and anchor
      (\?#{Twitter::TwitterText::Regex[:valid_url_query_chars]}*#{Twitter::TwitterText::Regex[:valid_url_query_ending_chars]})? #   $7 Query String
    )
  }iox
```

Then it extracts URLs with:

```ruby
# https://github.com/mastodon/mastodon/blob/main/app/services/fetch_link_card_service.rb#L73
def parse_urls
  urls = if @status.local?
            @status.text.scan(URL_PATTERN).map { |array| Addressable::URI.parse(array[1]).normalize }
          else
            document = Nokogiri::HTML(@status.text)
            links = document.css('a')

            links.filter_map { |a| Addressable::URI.parse(a['href']) unless skip_link?(a) }.filter_map(&:normalize)
          end

  urls.reject { |uri| bad_url?(uri) }.first
end
```

#### **TagSearchService**

Here is a nice and easy-to-read method in [`TagSearchService`](https://github.com/mastodon/mastodon/blob/main/app/services/tag_search_service.rb#L3)

```ruby
# https://github.com/mastodon/mastodon/blob/main/app/services/tag_search_service.rb#L4

def call(query, options = {})
  @query   = query.strip.delete_prefix('#')
  @offset  = options.delete(:offset).to_i
  @limit   = options.delete(:limit).to_i
  @options = options

  results   = from_elasticsearch if Chewy.enabled?
  results ||= from_database

  results
end
```

### Views

The views have the following structure:

* They are written using `HAML`
    
* The contains in the HAML file some content\_for header or open graph
    
* And they call at the end `shared/web_app`
    

Example from the `Home#index`:

```haml
- # https://github.com/mastodon/mastodon/blob/main/app/views/home/index.html.haml#L1
- content_for :header_tags do
  - unless request.path == '/'
    %meta{ name: 'robots', content: 'noindex' }/

  = render partial: 'shared/og'

= render 'shared/web_app'
```

Where the [`web_app`](https://github.com/mastodon/mastodon/blob/main/app/views/shared/_web_app.html.haml#L1) will do something like this:

* Preload some assets based on user sign-in
    
* [Render](https://github.com/mastodon/mastodon/blob/main/app/helpers/application_helper.rb#L192) by using InitialStatePresenter as JSON in a Script tag
    
* Include `javascript_pack_tag` for `application`
    

I will not go into this review into the Javascript part as this is focused on Ruby and Rails.

## Testing

It uses RSPec for testing.

In general, the tests are pretty straightforward, with very few custom matches defined.

The tests are easy to find as the folder inside `spec` matches the folder in `app`.

The coverage is a bit hard to guess from the test folder structure:

* not all controllers have a corresponding test in controller specs
    
* , not all models have a corresponding test in model specs
    

Just looking at some numbers (as I don't have time to understand the entire codebase) here is a comparison of the number of files in the app folder vs the number of files in the corresponding specs folder.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701922888325/d3168ce8-8296-44e8-a924-4c157a917a58.png align="center")

### Generating data

It uses [`Fabricator`](https://fabricationgem.org) gem to generate test data. Here is an example of generating a User:

```ruby
Fabricator(:user) do
  account      { Fabricate.build(:account, user: nil) }
  email        { sequence(:email) { |i| "#{i}#{Faker::Internet.email}" } }
  password     '123456789'
  confirmed_at { Time.zone.now }
  current_sign_in_at { Time.zone.now }
  agreement true
end
```

and then it can be called as `Fabricator(:user)`

### Testing views

Probably the React part is tested in some other specific way. I will not dig into that. But here is a test for open graph tags included in the status view:

```ruby
# https://github.com/mastodon/mastodon/blob/main/spec/views/statuses/show.html.haml_spec.rb#L5
describe 'statuses/show.html.haml', :without_verify_partial_doubles do
  before do
    allow(view).to receive_messages(api_oembed_url: '', show_landing_strip?: true, site_title: 'example site', site_hostname: 'example.com', full_asset_url: '//asset.host/image.svg', current_account: nil, single_user_mode?: false)
    allow(view).to receive(:local_time)
    allow(view).to receive(:local_time_ago)
    assign(:instance_presenter, InstancePresenter.new)
  end

  it 'has valid opengraph tags' do
    alice  = Fabricate(:account, username: 'alice', display_name: 'Alice')
    status = Fabricate(:status, account: alice, text: 'Hello World')
    Fabricate(:media_attachment, account: alice, status: status, type: :video)

    assign(:status, status)
    assign(:account, alice)
    assign(:descendant_threads, [])

    render

    header_tags = view.content_for(:header_tags)

    expect(header_tags).to match(/<meta content=".+" property="og:title">/)
    expect(header_tags).to match(/<meta content="article" property="og:type">/)
    expect(header_tags).to match(/<meta content=".+" property="og:image">/)
    expect(header_tags).to match(%r{<meta content="http://.+" property="og:url">})
  end
# code omitted ...
end
```

### Testing Rack::Attack

This is done via shared\_examples:

```ruby
# https://github.com/mastodon/mastodon/blob/main/spec/config/initializers/rack/attack_spec.rb#L5

escribe Rack::Attack, type: :request do
  def app
    Rails.application
  end

  shared_examples 'throttled endpoint' do
    before do
      # Rack::Attack periods are not rolling, so avoid flaky tests by setting the time in a way
      # to avoid crossing period boundaries.

      # The code Rack::Attack uses to set periods is the following:
      # https://github.com/rack/rack-attack/blob/v6.6.1/lib/rack/attack/cache.rb#L64-L66
      # So we want to minimize `Time.now.to_i % period`

      travel_to Time.zone.at(counter_prefix * period.seconds)
    end

    context 'when the number of requests is lower than the limit' do
      before do
        below_limit.times { increment_counter }
      end

      it 'does not change the request status' do
        expect { request.call }.to change { throttle_count }.by(1)

        expect(response).to_not have_http_status(429)
      end
    end

    def throttle_count
      described_class.cache.read("#{counter_prefix}:#{throttle}:#{remote_ip}") || 0
    end
 # code omitted ..
end
```

and the shared example will be used like this:

```ruby
context 'when accessed through the website' do
  let(:throttle) { 'throttle_sign_up_attempts/ip' }
  let(:limit)  { 25 }
  let(:period) { 5.minutes }
  let(:request) { -> { post path, headers: { 'REMOTE_ADDR' => remote_ip } } }

  context 'with exact path' do
    let(:path) { '/auth' }

    it_behaves_like 'throttled endpoint'
  end
# code omitted ..
end
```

## Conclusion

Mastodon open-source web app is a well-structured code base, easy to navigate (when focusing on the Ruby on Rails part). There is a good structure for testing and it probably covers most of the important functions, but there are still some models and controllers left out of testing that could be covered.

The code design is simple and easy to understand. The app tried to have slim controllers and pretty well-contained models while delegating more complex business logic to Service objects and simple Ruby objects. This makes the existing tests simple to follow and understand.

I think it could be a good pick for someone who wants to contribute. There are useful pieces of code that can be used as inspiration for other parts.