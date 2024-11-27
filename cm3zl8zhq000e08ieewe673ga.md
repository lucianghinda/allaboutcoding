---
title: "How to make a small pulsating animation"
seoTitle: "Create a Simple Pulsating Animation"
seoDescription: "Create subtle pulsating animations using Tailwind for enhanced menu item visibility. Learn how to build reusable UI components with Phlex"
datePublished: Wed Nov 27 2024 07:52:44 GMT+0000 (Coordinated Universal Time)
cuid: cm3zl8zhq000e08ieewe673ga
slug: how-to-make-a-small-pulsating-animation
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1732693911789/d677943e-eee7-4cee-aa01-3a2f03f487eb.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1732693921407/6f9fc62d-7801-4c31-bbc6-95174fc60746.png
tags: ruby, ruby-on-rails, animation, tailwind-css

---

The other day, I added a deals page to the [GoodEnoughTesting](https://goodenoughtesting.com) website and considered making the menu item a bit more visible. I wanted it to pulsate just a bit, without being too much.

Tailwind already has an animation called `pulse`, but I wanted something else. I wanted something that would scale a bit and then come back.

This is what I implemented:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732680863800/76416dd8-dec7-4258-8575-15aff09664e6.gif align="center")

First, I added the following in the `application.tailwind.css` where I define a 3 keyframes, and I scale it just a bit over at 50% (or the middle frame):

```css
@keyframes pulsate {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.10);
  }
  100% {
    transform: scale(1);
  }
}
```

Then I extended Tailwind to include this as animation, by adding the following in the `config/tailwind.config.js`:

```diff
module.exports = {
  theme: {
    extend: {
+      animation: {
+        'pulsate': 'pulsate 2s ease-in-out infinite',
+      }
    },
  },
}
```

And now I can use it in the header menu:

```xml
<a href="" class="motion-safe:animate-pulsate"> 
  A pulsating link 
</a>
```

I used `motion-safe` modifier to respected the choice of the user if they requested reduced motion.

## Extracting a component

I am using [Phlex](https://www.phlex.fun) when building the UI for the [GoodEnoughTesting](https://goodenoughtesting.com) so I want to extract the menu item to a series of components that I can re-use when needed.

First I created a `Menu::ItemComponent`:

```ruby
module Menu
  class ItemComponent < ApplicationComponent
    include Phlex::Rails::Helpers::LinkTo

    CLASS = <<~DEFAULT
      inline-block
      no-underline
      font-bold text-base uppercase leading-6 antialiased
      x-2.5 py-2 px-2
      align-baseline
      hover:bg-blue-200 hover:rounded-lg
    DEFAULT

    def initialize(url:, text: nil, pulsate: false, text_color: "", **options)
      @text = text
      @url = url
      @options = options
      @pulsate = pulsate
      @text_color = text_color
    end

    def template(&)
      if @text.present?
        link_to(@text, @url, class: css_rules)
      else
        link_to(@url, class: css_rules, &)
      end
    end

    private

      def css_rules   = "#{CLASS} #{pulsate_css} #{@text_color}"
      def pulsate_css = (@pulsate ? "motion-safe:animate-pulsate" : "")
  end
end
```

And then I created simple versions of this component.

`Menu::PrimaryComponent` is the normal menu item, with a dark gray color:

```ruby
module Menu
  class PrimaryComponent < ItemComponent
    def initialize(url:, text: nil, **options)
      super(url: url, text: text, pulsate: false, text_color: "text-gray-800", **options)
    end
  end
end
```

`Menu::BlueComponent` is a menu item with the blue text:

```ruby
module Menu
  class BlueComponent < ItemComponent
    def initialize(url:, text: nil, **options)
      super(url: url, text: text, pulsate: false, text_color: "text-blue-700", **options)
    end
  end
end
```

`Menu::PulsatingComponent` is the one that is setting the `pulsate` attribute:

```ruby
module Menu
  class PulsatingComponent < ItemComponent
    def initialize(url:, text: nil, **options)
      super(url: url, text: text, pulsate: true, text_color: "text-blue-700", **options)
    end
  end
end
```

And then at the end I combine them inside an ERB file like this:

```erb
<%= render Menu::PrimaryComponent.new(url: page_path("articles"), text: "Articles and News") %>
<%= render Menu::PulsatingComponent.new(url: page_path("deals"), text: "Deals", pulsate: true) %>
<%= render Menu::PrimaryComponent.new(url: page_path("about"), text: "About") %>
<%= render Menu::PrimaryComponent.new(url: page_path("contact"), text: "Contact") %>
<%= render Menu::BlueComponent.new(url: participants_sign_in_path, text: "Login") %>
```

---

If you like this article:

👐 Join my live workshop about [**goodenoughtesting.com**](http://goodenoughtesting.com/) **\- to learn test design techniques for writing effective tests**

👉 Join my [**Short Ruby Newsletter**](https://newsletter.shortruby.com/) for weekly Ruby updates from the community and visit [**rubyandrails.info**](http://rubyandrails.info/)**, a directory with learning content about Ruby.**

🤝 Let's connect on [Bluesky](https://bsky.app/profile/lucianghinda.com), [Ruby.social](http://ruby.social/), [Linkedin](https://linkedin.com/in/lucianghinda), [Twitter](https://x.com/lucianghinda) where I post mostly about Ruby and Ruby on Rails.

🎥 Follow me on [**my YouTube channel**](https://www.youtube.com/@shortruby) for short videos about Ruby/Rails