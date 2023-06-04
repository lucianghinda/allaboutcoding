---
title: "Stay up to date with new Ruby or Rails features"
seoTitle: "How to stay up to date with Ruby and Rails"
seoDescription: "Practical ideas about how to stay up to date with the newest Ruby and Rails features while also working on your projects."
datePublished: Wed May 31 2023 08:26:39 GMT+0000 (Coordinated Universal Time)
cuid: clibg1hka01hfvrnv5zwpbm4z
slug: stay-up-to-date-with-new-ruby-or-rails-features
tags: ruby, ruby-on-rails, developer, coding

---

Inspired by this post from Ruby for all:

%[https://twitter.com/rubyforall/status/1660641879170625536] 

Here is one possible way to stay up to date with the latest Ruby features or latest Rails or any other framework changes.

### **Experiment with a new feature**

Go to [https://rubyreferences.github.io/rubychanges](https://rubyreferences.github.io/rubychanges) and then pick one of the changes introduced in Ruby 3.0, 3.1 or 3.2. Pick the version that matches one of the current projects you are actively working on.

After choosing one new feature or change, try for a couple of weeks to write code using that feature. Don't force it but also try to reach for it more often.

Without playing with a new language feature it is very hard to think about cases where it fits and cases where maybe it does not fit.

Yes, sometimes your colleagues might not accept that in a PR. What I do sometimes is replace the new feature with the old code before submitting the PR.

You might think, why do this? Because it is important to work with new features in real projects and try to write real solutions.

### Draft a proposal to adopt a new feature

After playing around with a new feature, try to summarise your learnings into a draft proposal to adopt this new feature.

The draft should have at least 3 sections:

* **Why this feature?** Where you should describe what this feature brings on, what solutions will it open for you and your team
    
* **When to use it.** Here you should describe examples of good usage for the new feature. Where does it fit best, and how the code might look when using it
    
* **When not to use it.** Here you should describe examples of bad usage of the new feature. Where do you think it should not be used and why?
    

Send this to your colleagues or publish them on your blog. It will be great to contribute this way back to the Ruby community.

### Do the same with new Rails features or Hanami 2 or Roda or any other framework

You can do the same with Rails or Hanami 2 or Roda or any other web framework.

1. Go to their Github/Gitlab project
    
2. Checkout the latest PRs merges to `main`
    
3. Change your Gemfile to point to the `main` branch
    
4. Choose one PR that was merged and try to use the changes/features proposed there to implement a solution
    

## Share your learnings

It would be great to share your insights with the public.

Write a blog post where you talk about what you learned, how you used a new feature, what problems you encountered and when and how you think it should be used.

If you don't have a blog, then write a public gist and add it to Github.

In case you cannot do any of the above just share it via social media.

It will of course be great if you can combine all: write a blog post and then share it via social media.

---
Enjoyed this article? 

Join my **[Short Ruby News](https://shortruby.com) newsletter** for weekly Ruby updates. Also, check out my co-authored **book**, **[LintingRuby](https://lintingruby.com)**, for insights on automated code checks. For more Ruby **learning resources**, visit **[rubyandrails.info](https://rubyandrails.info)**.