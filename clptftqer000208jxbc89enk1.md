---
title: "An Endless Method Use Case"
datePublished: Wed Dec 06 2023 07:18:25 GMT+0000 (Coordinated Universal Time)
cuid: clptftqer000208jxbc89enk1
slug: an-endless-method-use-case
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1701847030088/5f1a959f-772c-41c7-a1d9-31b0f2b3ac39.png
tags: refactoring, ruby, opensource, ruby-on-rails

---

While preparing my article for the series that I started about [Open Source Ruby](https://allaboutcoding.ghinda.com/series/open-source-ruby), I found an example of code that I think could benefit from using the endless method.

Here is a video version of this article:

%[https://www.youtube.com/watch?v=nTHL0dvqmI0] 

or continue reading for the text version 👇

## The original code

The original code is based on Mastodon [ApplicationController](https://github.com/mastodon/mastodon/blob/main/app/controllers/application_controller.rb#L3):

```ruby
# https://github.com/mastodon/mastodon/blob/main/app/controllers/application_controller.rb#L3

class ApplicationController < ActionController::Base
  # other code ...

  def forbidden
    respond_with_error(403)
  end

  def not_found
    respond_with_error(404)
  end

  def gone
    respond_with_error(410)
  end

  def unprocessable_entity
    respond_with_error(422)
  end

  def not_acceptable
    respond_with_error(406)
  end

  def bad_request
    respond_with_error(400)
  end

  def internal_server_error
    respond_with_error(500)
  end

  def service_unavailable
    respond_with_error(503)
  end

  def too_many_requests
    respond_with_error(429)
  end
  # other code ...
end
```

## Refactoring

### Use the endless method

```ruby
# https://github.com/mastodon/mastodon/blob/main/app/controllers/application_controller.rb#L3

class ApplicationController < ActionController::Base
  # other code ...

  def forbidden = respond_with_error(403)

  def not_found = respond_with_error(404)

  def gone = respond_with_error(410)

  def unprocessable_entity = respond_with_error(422)

  def not_acceptable = respond_with_error(406)

  def bad_request = respond_with_error(400)

  def internal_server_error = respond_with_error(500)

  def service_unavailable = respond_with_error(503)

  def too_many_requests = respond_with_error(429)

  # other code ...
end
```

In this specific case, the endless method is the best to be used here.

It is almost like aliasing the same method but giving it different names based on the parameters passed.

Let's take a look at a side-by-side comparison:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701846368932/bd86a9e4-0aa1-4b4b-a4c2-00600c539978.png align="center")

### Extract to a concern

I think we can take this one step further and extract it all to a concern or simple module if you want to:

```ruby
module RespondWithError
  extend ActiveSupport::Concern

  private

  def forbidden = respond_with_error(403)

  def not_found = respond_with_error(404)

  def gone = respond_with_error(410)

  def unprocessable_entity = respond_with_error(422)

  def not_acceptable = respond_with_error(406)

  def bad_request = respond_with_error(400)

  def internal_server_error = respond_with_error(500)

  def service_unavailable = respond_with_error(503)

  def too_many_requests = respond_with_error(429)

  def respond_with_error(code)
    respond_to do |format|
      format.any  { render "errors/#{code}", layout: 'error', status: code, formats: [:html] }
      format.json { render json: { error: Rack::Utils::HTTP_STATUS_CODES[code] }, status: code }
    end
  end
end
```

This way all behavior about responding with error status code is contained in a single file. Adding or changing this behaviour could be made in this one single place.

### Group together by status code

Based on the reply from [Russell Garner](https://ruby.social/@rgarner@mastodon.social/111533461326388462) and [Bradley Schaefer](https://ruby.social/@soulcutter/111533488000059521) we can refactor this more by removing new lines between endless methods and grouping them by status:

```ruby
module RespondWithError
  extend ActiveSupport::Concern

  private

  # Client error responses 4xx
  def bad_request           = respond_with_error(400)
  def forbidden             = respond_with_error(403)
  def not_found             = respond_with_error(404)
  def not_acceptable        = respond_with_error(406)
  def gone                  = respond_with_error(410)
  def unprocessable_entity  = respond_with_error(422)
  def too_many_requests     = respond_with_error(429)

  # Server error responses 5xx
  def internal_server_error = respond_with_error(500)
  def service_unavailable   = respond_with_error(503)

  def respond_with_error(code)
    respond_to do |format|
      format.any  { render "errors/#{code}", layout: 'error', status: code, formats: [:html] }
      format.json { render json: { error: Rack::Utils::HTTP_STATUS_CODES[code] }, status: code }
    end
  end
end
```

## Read more about the endless method

I strongly recommend this article from Victor Shepelev about the endless method:

[https://zverok.substack.com/p/useless-ruby-sugar-endless-one-line](https://zverok.substack.com/p/useless-ruby-sugar-endless-one-line)

You will find there more cases and examples about when to use and when not to use this construct.

---

Enjoyed this article?

Join my [**Short Ruby News**](https://shortruby.com/) **newsletter** for weekly Ruby updates from the community. For more Ruby **learning resources**, visit [**rubyandrails.info**](http://rubyandrails.info). You can also find me on [**Ruby.social**](https://ruby.social/@lucian) or [**Linkedin**](https://linkedin.com/in/lucianghinda) or [**Twitter**](https://x.com/lucianghinda) where I post mostly about Ruby and Rails. Subscribe to [**my YouTube channel**](https://www.youtube.com/@shortruby) for short videos about Ruby.

---

## Updates

* 2023-12-06: Added a new refactoring variant based on replies to [my share on Ruby.social](https://ruby.social/@lucian/111533361426467938) - see `Group together by status code`