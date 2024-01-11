---
title: "Ruby Open Source: chatwoot"
seoTitle: "Explore Chatwoot: Open Source Ruby on Rails customer engagement app"
seoDescription: "Explore Chatwoot, an open-source Ruby suite competing with Intercom, Zendesk, Salesforce; boost customer support efficiently"
datePublished: Fri Nov 10 2023 06:51:20 GMT+0000 (Coordinated Universal Time)
cuid: clos9erlp000t09kzdeug1xvd
slug: ruby-open-source-chatwoot
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1699418020706/623fc755-3f2a-410a-b77d-2e76d7980c4c.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1699418168201/1f4e83f9-c156-47c7-ac27-417e414db421.png
tags: ruby, opensource, ruby-on-rails, coding, reviews

---

Continuing the series about open-source Ruby with [chatwoot](https://www.chatwoot.com)

As a reminder, this is a 30-minute review so this is a high-level overview and there might be things that I missed or that I misunderstood.

## The product

[chatwoot](https://github.com/chatwoot/chatwoot) is an *"Open-source customer engagement suite, an alternative to Intercom, Zendesk, Salesforce Service Cloud etc"*

[![What is chatwoot from their website](https://cdn.hashnode.com/res/hashnode/image/upload/v1699329500672/3fed05ce-d1c7-486a-bf15-c33b50394a51.png align="center")](https://chatwoot.com)

The interface looks like this ([source their website](https://www.chatwoot.com))

[![an example of chatwood main interface](https://cdn.hashnode.com/res/hashnode/image/upload/v1699329651363/7faa599d-775d-4be2-b3fa-d1ec4cf9ae3b.png align="center")](https://www.chatwoot.com)

They were part of [YCombinator batch in 2021](https://www.ycombinator.com/companies/chatwoot).

## Open Source

### Repository and License

The repository is on Github at [https://github.com/chatwoot/chatwoot](https://github.com/chatwoot/chatwoot) and it is open sourced with [a license that seems a variant of MIT license](https://github.com/chatwoot/chatwoot/blob/develop/LICENSE):

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699330118730/f1d3e6ee-101b-4b8a-bfca-2644146b7975.png align="center")

### Ruby and Rails version

At the moment of writing this article (November 2023), it runs on [Ruby 3.2.2](https://github.com/chatwoot/chatwoot/blob/develop/.ruby-version#L1) and [Rails 7.0.8](https://github.com/chatwoot/chatwoot/blob/develop/Gemfile#L7)

### Architecture

**Backend**

* It uses Rails to create a REST API with [jbuilder](https://github.com/rails/jbuilder) as the serializer
    

**Frontend**

* It uses Vue for the front end with Tailwind CSS. You can explore the package.json [here](https://github.com/chatwoot/chatwoot/blob/develop/package.json)
    
* It also uses Administrate gem as an admin dashboard
    

**Background processing queue**

* Sidekiq with Redis
    

**Database:**

* PostgreSQL
    

### Stats

Running `rails stats` returned the following:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699330889681/57df088e-0ad1-4d5f-b5ff-900528110363.png align="center")

As this project uses Vue on the front, here is the output of running an extension in VScode called Code Counter:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699331122243/cc7b39cd-d5d4-4de3-9376-a0d2f2973a32.png align="center")

Not sure how it does all these calculations but it seems the application has quite some lines of code in JS (JavaScript and Vue files together).

### Style Guide

They use Rubocop with some custom settings added in their [.rubocop.yml file](https://github.com/chatwoot/chatwoot/blob/develop/.rubocop.yml):

```yaml
require:
  - rubocop-performance
  - rubocop-rails
  - rubocop-rspec
```

Among the cops that have their default settings changed:

* RSpec/ExampleLength with max=25
    
* Style/FrozenStringLiteralComment with enabled=false
    
* Style/OpenStructUse with enabled=false
    
* Style/GlobalVars are allowed in redis and rack\_attack initializers and in `lib/global_config.rb`
    
* Style/ClassVars are allowed in one file `app/services/email_templates/db_resolver_service.rb`
    
* Style/HashSyntax has the shorthand syntax disabled
    
* Naming/VariableNumber is disabled
    

and there are more there.

### Storage, Persistence and in-memory storage

As a database it uses PostgreSQL.

It defines two global variables for Redis:

```ruby
# Alfred
# Add here as you use it for more features
# Used for Round Robin, Conversation Emails & Online Presence
$alfred = ConnectionPool.new(size: 5, timeout: 1) do
  redis = Rails.env.test? ? MockRedis.new : Redis.new(Redis::Config.app)
  Redis::Namespace.new('alfred', redis: redis, warning: true)
end

# Velma : Determined protector
# used in rack attack
$velma = ConnectionPool.new(size: 5, timeout: 1) do
  config = Rails.env.test? ? MockRedis.new : Redis.new(Redis::Config.app)
  Redis::Namespace.new('velma', redis: config, warning: true)
end
```

Then these global variables are used in other places in the code:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699343659174/7f687fa8-4cb2-4705-8c12-57b5b7b4429e.png align="center")

### Gems used

Here is a selection from all the gems they use:

* [act\_as\_taggable\_on](https://github.com/mbleigh/acts-as-taggable-on) - *"A tagging plugin for Rails applications that allows for custom tagging along dynamic contexts"*
    
* [attr\_extras](https://github.com/barsoom/attr_extras) - *"Takes some boilerplate out of Ruby with methods like attr\_initialize"*
    
* [hashie](https://github.com/hashie/hashie) - *"Hashie is a growing collection of tools that extend Hashes and make them more useful"*
    
* [responders](https://github.com/heartcombo/responders) - *"A set of Rails responders to dry up your application"*
    
* [telephone\_number](https://github.com/mobi/telephone_number) - *"TelephoneNumber is global phone number validation gem based on Google's libphonenumber library"*
    
* [valid\_email2](https://github.com/micke/valid_email2) - "*Validate emails with the help of the mail gem instead of some clunky regexp. Aditionally validate that the domain has a MX record. Optionally validate against a static list of disposable email services. Optionally validate that the email is not subaddressed (RFC5233)"*
    
* [flag\_shih\_tzu](https://github.com/pboling/flag_shih_tzu) - *"This gem lets you use a single integer column in an ActiveRecord model to store a collection of boolean attributes (flags). Each flag can be used almost in the same way you would use any boolean attribute on an ActiveRecord object"*
    
* [haikunator](https://github.com/usmanbashir/haikunator) - *"Generate Heroku-like memorable random names to use in your apps or anywhere else."*
    
* [commonmarker](https://github.com/gjtorikian/commonmarker) - *"Ruby wrapper for Rust's comrak crate. It passes all of the CommonMark test suite, and is therefore spec-complete. It also includes extensions to the CommonMark spec as documented in the GitHub Flavored Markdown spec, such as support for tables, strikethroughs, and autolinking"*
    
* [down](https://github.com/janko/down) -*"Down is a utility tool for streaming, flexible and safe downloading of remote files. It can use open-uri + Net::HTTP, http.rb, HTTPX, or wget as the backend HTTP library"*
    
* [gmail\_xoauth](https://github.com/nfo/gmail_xoauth) - *"Get access to Gmail IMAP and SMTP via OAuth2 and OAuth 1.0a, using the standard Ruby Net libraries. The gem supports 3-legged OAuth, and 2-legged OAuth for Google Apps Business or Education account owners"*
    
* [csv-safe](https://github.com/zvory/csv-safe) - *"This gem decorates the built in CSV library to prevent CSV injection attacks. Wherever you would use CSV in your code, use CSVSafe. The gem will encode your fields in UTF-8"*
    
* [hairtrigger](https://github.com/jenseng/hair_trigger) - *"lets you create and manage database triggers in a concise, db-agnostic, Rails-y way. You declare triggers right in your models in Ruby, and a simple rake task does all the dirty work for you"*
    
* [wisper](https://github.com/krisleech/wisper) - *"A micro library providing Ruby objects with Publish-Subscribe capabilities"*
    
* [groupdate](https://github.com/ankane/groupdate) - *"The simplest way to group by: day, week, hour of the day, and more"*
    
* [html2text](https://github.com/soundasleep/html2text_ruby) - *"a very simple gem that uses DOM methods to convert HTML into a format similar to what would be rendered by a browser - perfect for places where you need a quick text representation"*
    
* [informers](https://github.com/ankane/informers) - *"State-of-the-art natural language processing for Ruby: Sentiment analysis, Question answering, Named-entity recognition, Text generation"*
    
* [climate\_control](https://github.com/thoughtbot/climate_control) - *"Climate Control modifies environment variables only within the context of the block, ensuring values are managed properly and consistently"*
    

### Design Patterns

The "app" directory has the following sub-folders that are not the ones included by default in Rails:

* actions
    
* builders
    
* dashboards
    
* dispatchers
    
* drops
    
* fields
    
* finders
    
* listeners
    
* mailboxes
    
* policies
    
* presenters
    
* services
    
* workers
    

Here are some examples of some patterns from the codebase:

#### Actions

Inside actions, some objects define the `perform` method where they execute a series of steps:

```ruby
class SomeAction
  pattr_initialize [:user, :base]

  def perform
    ActiveRecord::Base.transaction do
      action1
      action2
      # ...
    end
  end
end
```

Where `pattr_initialize` comes from [attr\_extras](https://github.com/barsoom/attr_extras?tab=readme-ov-file#pattr_initialize) gem and it will create an initializer with those variables and declare them as private.

**Builders**

These have the same interface where they define `perform` and it seems to be a flavor of Builder Pattern maybe with a service object where the builder will create or find one or more records.

Here is an example of the [`AccountBuilder`](https://github.com/chatwoot/chatwoot/blob/abbb4180ea9fb1da007a659ef8cb28188321c641/app/builders/account_builder.rb#L7) that will create an account and then create a user and link these two together.

```ruby
class AccountBuilder
  include CustomExceptions::Account
  pattr_initialize [:account_name!, :email!, :confirmed, :user, :user_full_name, :user_password, :super_admin, :locale]

  def perform
    if @user.nil?
      validate_email
      validate_user
    end
    ActiveRecord::Base.transaction do
      @account = create_account
      @user = create_and_link_user
    end
    [@user, @account]
  rescue StandardError => e
    puts e.inspect
    raise e
  end

  # ... more methods
end
```

#### Controllers

Take a look at the `app/controllers/api` there is a versioning of the API using namespaces (`app/controllers/api/v1` , `app/controllers/api/v2` )

The controllers appears to be slim calling other objects and usually returning a JSON represented in views via `.json.builder` files.

#### Dashboards

Here they define the [Administrate](https://github.com/thoughtbot/administrate) custom dashboards

#### Dispatchers

They implement the [`Wisper::Publisher`](https://github.com/krisleech/wisper) module to dispatch events. [Here](https://github.com/chatwoot/chatwoot/blob/abbb4180ea9fb1da007a659ef8cb28188321c641/app/dispatchers/async_dispatcher.rb#L1) is an example of an async dispatcher using a job:

```ruby
# https://github.com/chatwoot/chatwoot/blob/develop/app/dispatchers/async_dispatcher.rb

class AsyncDispatcher < BaseDispatcher
  def dispatch(event_name, timestamp, data)
    EventDispatcherJob.perform_later(event_name, timestamp, data)
  end

  def publish_event(event_name, timestamp, data)
    event_object = Events::Base.new(event_name, timestamp, data)
    publish(event_object.method_name, event_object)
  end

  def listeners
    [
      CampaignListener.instance,
      CsatSurveyListener.instance,
      HookListener.instance,
      InstallationWebhookListener.instance,
      NotificationListener.instance,
      ReportingEventListener.instance,
      WebhookListener.instance,
      AutomationRuleListener.instance
    ]
  end
end
```

#### Finders

Finders are another patterns used here which takes care of querying the DB while taking care of filters or other conditions coming from `params` in controllers. They usually define a `perform` method and the initializer accepts a `params` to get the params from the controller.

Here is an [example](https://github.com/chatwoot/chatwoot/blob/develop/app/finders/conversation_finder.rb#L1):

```ruby
# https://github.com/chatwoot/chatwoot/blob/develop/app/finders/conversation_finder.rb#L1
class ConversationFinder
  attr_reader :current_user, :current_account, :params

  # params
  # assignee_type, inbox_id, :status

  def initialize(current_user, params)
    @current_user = current_user
    @current_account = current_user.account
    @params = params
  end

  def perform
   # calls to compose the response from other methods 
  end

  private

  # other methods ...

  def find_all_conversations
    @conversations = current_account.conversations.where(inbox_id: @inbox_ids)
    filter_by_conversation_type if params[:conversation_type]
    @conversations
  end

  def filter_by_assignee_type
    case @assignee_type
    when 'me'
      @conversations = @conversations.assigned_to(current_user)
    when 'unassigned'
      @conversations = @conversations.unassigned
    when 'assigned'
      @conversations = @conversations.assigned
    end
    @conversations
  end

  def filter_by_conversation_type
    case @params[:conversation_type]
    when 'mention'
      conversation_ids = current_account.mentions.where(user: current_user).pluck(:conversation_id)
      @conversations = @conversations.where(id: conversation_ids)
    when 'participating'
      @conversations = current_user.participating_conversations.where(account_id: current_account.id)
    when 'unattended'
      @conversations = @conversations.unattended
    end
    @conversations
  end

  def filter_by_query
    return unless params[:q]

    allowed_message_types = [Message.message_types[:incoming], Message.message_types[:outgoing]]
    @conversations = conversations.joins(:messages).where('messages.content ILIKE :search', search: "%#{params[:q]}%")
                                  .where(messages: { message_type: allowed_message_types }).includes(:messages)
                                  .where('messages.content ILIKE :search', search: "%#{params[:q]}%")
                                  .where(messages: { message_type: allowed_message_types })
  end

  # some other methods ... 
end
```

#### Listeners

The listeners are implemented using the `Singleton` interface and all inherit from the `BaseListener` and usually will do an action like calling a Job or Builder or some other object when the event is received.

#### Mailboxes

This could be a good repo if you want to see how to read and process inbound emails. Take a look at [`ApplicationMailbox`](https://github.com/chatwoot/chatwoot/blob/develop/app/mailboxes/application_mailbox.rb#L1https://github.com/chatwoot/chatwoot/blob/develop/app/mailboxes/application_mailbox.rb#L1) which it matches either a reply to a conversation or a new conversation received via a channel:

```ruby
class ApplicationMailbox < ActionMailbox::Base
  include MailboxHelper

  # Last part is the regex for the UUID
  # Eg: email should be something like : reply+6bdc3f4d-0bec-4515-a284-5d916fdde489@domain.com
  REPLY_EMAIL_UUID_PATTERN = /^reply\+([0-9a-f]{8}\b-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-\b[0-9a-f]{12})$/i
  CONVERSATION_MESSAGE_ID_PATTERN = %r{conversation/([a-zA-Z0-9-]*?)/messages/(\d+?)@(\w+\.\w+)}

  # routes as a reply to existing conversations
  routing(
    ->(inbound_mail) { reply_uuid_mail?(inbound_mail) || in_reply_to_mail?(inbound_mail) } => :reply
  )

  # routes as a new conversation in email channel
  routing(
    ->(inbound_mail) { EmailChannelFinder.new(inbound_mail.mail).perform.present? } => :support
  )
# ... more methods
end
```

#### Mailers

The Mailers are implemented using [liquid](https://shopify.github.io/liquid/) gem. The logic to decide how to deliver a reply based on the inbox\_type can be found in the [`ConversationReplymailerHelper`](https://github.com/chatwoot/chatwoot/blob/develop/app/mailers/conversation_reply_mailer_helper.rb#L1) in a method that looks like this:

```ruby
# https://github.com/chatwoot/chatwoot/blob/develop/app/mailers/conversation_reply_mailer_helper.rb#L2
  def prepare_mail(cc_bcc_enabled)
    @options = {
      to: to_emails,
      from: email_from,
      reply_to: email_reply_to,
      subject: mail_subject,
      message_id: custom_message_id,
      in_reply_to: in_reply_to_email
    }

    if cc_bcc_enabled
      @options[:cc] = cc_bcc_emails[0]
      @options[:bcc] = cc_bcc_emails[1]
    end
    ms_smtp_settings
    set_delivery_method

    mail(@options)
  end
```

#### Models

Models are Active Record models with just the right amount of logic. They are not very big but they are also not slim.

#### Policies

They use [pundit](https://github.com/varvet/pundit) gem and thus this folder contains policies. Each method is simple and easy to understand.

#### Services

There are 63 files inside the services having a similar interface, defining a `perform` or `perform_reply` method.

#### Lib

There are over 60 fiels in the `/lib` folder. They don't use the same pattern and are doing various things. From defining some types or constants like [`Events::Types`](https://github.com/chatwoot/chatwoot/blob/develop/lib/events/types.rb#L3) to for example a client connecting to [`MicrosoftGraphAuth`](https://github.com/chatwoot/chatwoot/blob/develop/lib/microsoft_graph_auth.rb#L11)

### Testing

They use RSpec for testing with FactoryBot and some fixtures.

Looking a bit at the structure of some tests:

* They use nested `describe` or `describe` with nested `context`
    
* `let` and `let!`
    
* `subject`
    
* A limited number of `shared_examples` inside `models`
    

The tests appear to be simple with little setup in a before block and some `let!` and the test is self-contained.

## Conclusion

In conclusion, chatwoot is an open-source customer engagement suite with a well-structured codebase and a variety of design patterns. Most of them are trying to define a common interface (like `perform` method). The controllers and models are pretty close to vanilla Rails: slim controllers, with some logic in models, making use of callbacks.

It employs several gems and libraries to enhance its functionality but without changing too much the flavour of Rails.

The project uses Ruby 3.2.2, Rails 7.0.8, and Vue for its front end and also has a bit of ERB for the Administrate gem.

From the Ruby on Rails perspective it could be a project easy to grasp with a bit of mental load on the testing side where there is a mix of `let`, `subject` and `shared_examples`.

---

Enjoyed this article?

👉 Join my [**Short Ruby News**](https://shortruby.com/) **newsletter** for weekly Ruby updates from the community and visit [**rubyandrails.info**](http://rubyandrails.info)**,** a directory with learning content about Ruby.

👐 Subscribe to my Ruby and Ruby on rails courses over email at [learn.shortruby.com](https://learn.shortruby.com) - effortless learning anytime, anywhere

🤝 Let's connect on [**Ruby.social**](https://ruby.social/@lucian) or [**Linkedin**](https://linkedin.com/in/lucianghinda) or [**Twitter**](https://x.com/lucianghinda) where I post mainly about Ruby and Rails.

🎥 Follow me on [**my YouTube channel**](https://www.youtube.com/@shortruby) for short videos about Ruby