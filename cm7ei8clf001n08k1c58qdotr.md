---
title: "How to make sure you review your monkey patch when updating Ruby gems"
seoTitle: "Reviewing monkey patches when updating gems"
seoDescription: "Learn how to review and manage monkey patches when updating Ruby gems to ensure consistent functionality and maintain code quality"
datePublished: Fri Feb 21 2025 08:23:55 GMT+0000 (Coordinated Universal Time)
cuid: cm7ei8clf001n08k1c58qdotr
slug: how-to-make-sure-you-review-your-monkey-patch-when-updating-ruby-gems
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1740126169588/fe66b63a-423e-4a16-83b7-7013b222d0d0.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1740126202079/71999227-76ff-4401-9c73-067b35572761.png
tags: ruby, ruby-on-rails, updates

---

When I installed Writebook for [booklet.goodenoughtesting.com](https://booklet.goodenoughtesting.com), I monkey-patched some parts of it because I wanted to remove the creation of the first book. Later, I added some default meta tags. You can read about my approach in the article [“Overriding Methods in Ruby on Rails: A No-Code-Editing](https://allaboutcoding.ghinda.com/overriding-methods-in-ruby-on-rails-a-no-code-editing-approach) Approach.” You might monkey patch a gem or a part of Rails for different reasons.

I agree that monkey patching should be the last approach and used carefully. One of the most critical issues is that the code you create depends on the *structure* of the code you are monkey patching, thus voluntarily violating the contract that the gem's author offered.

I see monkey patching as a breach of contract with the author of a gem. In rare cases when this is necessary, the fix should be on the side that breached the contract. If you go down this route, then every time the gem is updated, you have to review the code of the gem and test your monkey patch to see if everything works as you expect it.

## How to automate the check of patched Gem version

Add the following code at the beginning of your monkey patch file.

```ruby
# This is an example about ActiveSupport

if Rails.env.local? || Rails.env.ci?
  if ActiveSupport.version > Gem::Version.new('8.0.0')
    raise "\n\nReview this monkey patch after upgrading \n" \
          "Path: #{__FILE__}"
  end
end
```

You can do the same if you have an initializer and you *temporary* added some settings that you know you want to remove or review after doing an update.

I found this approach in various projects I’ve worked on over the past couple of years. It can take different forms: sometimes only in CI, sometimes in a specific CI that checks compatibility, sometimes in tests, and other variations.

In case the gem does not define the version as an `Gem::Version` object and it might define it like this:

```ruby
module MyGem
  VERSION = '1.0.1'
end
```

Then you should create the comparison in the following way:

```diff
if Rails.env.local? || Rails.env.ci?
- if MyGem::Version > Gem::Version.new('8.0.0')
+  if Gem::Version.new(MyGem::VERSION) > Gem::Version.new('1.0.1')
    raise "\n\nReview this monkey patch after upgrading \n" \
          "Path: #{__FILE__}"
  end
end
```

You should always compare with `Gem::Version` object because it defines the method `<=>` in a way that takes into consideration major, minor and patch version. Here is how that method currently looks like:

```ruby
# Source: https://github.com/rubygems/rubygems/blob/master/lib/rubygems/version.rb#L360

  def <=>(other)
    return self <=> self.class.new(other) if (String === other) && self.class.correct?(other)

    return unless Gem::Version === other
    return 0 if @version == other.version || canonical_segments == other.canonical_segments

    lhsegments = canonical_segments
    rhsegments = other.canonical_segments

    lhsize = lhsegments.size
    rhsize = rhsegments.size
    limit  = (lhsize > rhsize ? lhsize : rhsize) - 1

    i = 0

    while i <= limit
      lhs = lhsegments[i] || 0
      rhs = rhsegments[i] || 0
      i += 1

      next      if lhs == rhs
      return -1 if String  === lhs && Numeric === rhs
      return  1 if Numeric === lhs && String  === rhs

      return lhs <=> rhs
    end

    0
  end
```

As you can notice it knows also how to compare with a `String` so you could probably write this code like this:

```ruby
if Rails.env.local? || Rails.env.ci?
  if Gem::Version.new("1.0.1") <= MyGem::VERSION
    raise "\n\nReview this monkey patch after upgrading \n" \
          "Path: #{__FILE__}"
  end
end
```

But I prefer to make sure that both of them are `Gem::Version` objects.

---

If you like this article:

👐 Interested in learning how to improve your developer testing skills? Join my live online workshop about [**goodenoughtesting.com**](http://goodenoughtesting.com/) **\- to learn test design techniques for writing effective tests**

👉 Join my [**Short Ruby Newsletter**](https://newsletter.shortruby.com/) for weekly Ruby updates from the community and visit [**rubyandrails.info**](http://rubyandrails.info/)**, a directory with learning content about Ruby.**

🤝 Let's connect on [**Bluesky**](https://bsky.app/profile/lucianghinda.com), [**Ruby.social**](http://ruby.social/)**,** [**Linkedin**](https://linkedin.com/in/lucianghinda)**,** [**Twitter**](https://x.com/lucianghinda) **where I post mostly about Ruby and Ruby on Rails.**

🎥 Follow me on [**my YouTube channel**](https://www.youtube.com/@shortruby) for short videos about Ruby/Rails