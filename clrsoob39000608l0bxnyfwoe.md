---
title: "The tech stack I choose to build my email courses project"
seoTitle: "Tech stack for my email courses project"
seoDescription: "Efficient email courses tech stack: Ruby, Rails, SQLite, litestack, Avo, Tailwind, ERBs, Phlex, Minitest, Sitepress, Debug, Propshaft, Hotwire, direnv..."
datePublished: Thu Jan 25 2024 03:57:46 GMT+0000 (Coordinated Universal Time)
cuid: clrsoob39000608l0bxnyfwoe
slug: the-tech-stack-i-choose-to-build-my-email-courses-project
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1706111082387/796f04f9-4e2a-4ba3-80ea-5e943d2a0c9e.png
tags: ruby, technology, ruby-on-rails, side-project

---

I am building a new project: [Ruby and Ruby on Rails Courses over Email](https://learn.shortruby.com).

In the beginning, I had to choose the tech stack for it. Here is an exploration of what criteria I considered, what I chose, and why.

## Criteria

When I think about the tech stack I want to use for this project, I have to consider the following:

**Time**

The most important part is time. This is a side project; thus, it receives a small part of my effort. This is a fixed allocation that I can put among spending time with family, having a full-time job, and taking time for myself.

I think about time in two ways:

* The time I have to build the project
    
* The time I will have to maintain the project
    

**Speed of change**

The second most important thing is being able to change. That includes releasing new features or reacting to users’ needs or wants.

What’s important here is the speed of change: How fast can I release something to production?

**Costs**

On the third place are costs: hosting, emailing, and additional services used to build the project.

I have not yet allocated a budget for this project but will do so over the next weeks. Having a budget will help to validate if the idea is *feasible* and *sustainable* over the long run.

## Choices

### Programming language: Ruby

I am using [**Ruby**](https://www.ruby-lang.org/en/). It is the language the I know best and it is also a language that allows me enough flexibility to write code that is easy to change and quick to adapt.

### Web framework: Ruby on Rails

I am using [**Ruby on Rails**](https://rubyonrails.org). I briefly considered Hanami 2 as it is fresh, and I wanted to pick it up, but considering the time and speed of change, there is not much room to learn a new framework. At the same time, Ruby on Rails comes with a lot of battery included, and that is what I need in this project where I plan to focus on the product and content.

### Persistence: SQLite

I am using **SQLite**. Ruby on Rails now has good support for SQLite in production, and [Stephen Margheim](https://fractaledmind.github.io) has a lot of easy-to-follow and to-the-point articles for configuring it. Choosing SQLite reduces the time allocated to maintenance, so it is an easy win.

I am not concerned with scalability as I don’t see this project needing more than a server.

I also added the [`Activerecord-enhancedsqlite3-adapter`](https://github.com/fractaledmind/activerecord-enhancedsqlite3-adapter) that adds support for generated columns, deferred foreign keys, PRAGMA tunning, and extensions loading.

For backup, I am using litestream gem, which makes working with [`litestream.io`](https://litestream.io) easier.

### Background running jobs

I use [litestack](https://github.com/oldmoe/litestack) gem that comes with everything, including Jobs, Caching, Search, and Metrics.

I am considering for jobs to move to [SolidQueue](https://github.com/basecamp/solid_queue) at some point.

### Admin: Avo

I am using [Avo](https://avohq.io) to build the Admin UI that I will use to manage the content. Avo looks excellent; it is modern and fast, and I don’t want to spend time building a UI for myself to manage the content.

It is already built, it has a feel of Railsy while configuring it, and it is flexible enough so that if I need more custom things, I can do them.

The best benefit is that I get time back to invest in the content instead of the development of an admin.

### CSS: Tailwind and TailwindUI

I am using [Tailwind](https://tailwindcss.com) because I used it professionally, so I know it a bit, and also because I bought [TailwindUI](https://tailwindui.com) where I can find ready-made components that I can copy/paste into my project if needed.

Rails 7 also comes with an already integrated Tailwind, and it works with import maps.

### Import maps

I like [Rails import maps](https://github.com/rails/importmap-rails), and I am picking other UI-related gems only if it works with import maps. That is because I don’t want to run `nodejs` or have any `node_modules` installed. It adds to maintenance, and I like to minimize maintenance tasks.

### Views: ERBs and Phlex

First, I will create the general views as HTML ERB files. This is because it is close to HTML, will have low maintenance, and can be changed quickly. It also helps with copying/pasting elements from TailwindUI without making too many changes.

I will organize (or extract) standard components by using [phlex](https://www.phlex.fun). Phlex is closer to my backend background, so I can keep writing Ruby while building UI elements. [`phlex-rails`](https://rubygems.org/gems/phlex-rails) has also a good integration with [`Lookbook`](https://lookbook.build/guide/components/phlex) and so I can easily preview my components.

How I use this mix is like this: the general structure of the page is done with HTML ERB, but the elements inside are mostly done with Phlex.

### Testing

I use [Minitest](https://guides.rubyonrails.org/testing.html#rails-meets-minitest), the default testing framework for Rails.

I like Minitest in my side projects for two main reasons:

* It comes default with Rails and runs fast
    
* It is not a DSL, but it is just Ruby code. So it will be easy to return to it and not spend time remembering a DSL.
    

There is one gem that I usually install for tests and that is [`minitest-stub-const`](https://github.com/adammck/minitest-stub-const). Stubbing constanta can also be done without this gem, but I kept using this gem.

### Static pages: Sitepress

For static pages, like [homepage](https://learn.shortruby.com), or [the blog section](https://learn.shortruby.com/blog) I am using [sitepress](https://sitepress.cc). It has an easy integration with Rails and helps move from static files to dynamic options easily. I also like that it has Frontmatter support and comes with a way to work with each page’s meta-data.

It is a maintenance and low configuration solution: Install it, write things in `.html.md` files, and all will work.

### Debugging: Debug gem

[Debug](https://github.com/ruby/debug) gem is now the default debugger gem for Rails, and I like it a lot.

A lot of development is happening, and new features are always added. I started moving away from `puts` debugging to real debugging, and I even use the debugger to inspect object states while I do code design.

As the `debug` gem is part of Ruby, I think it can be a safe bet for the future with little maintenance or overhead required. I am using it in all my projects.

### Asset pipeline: Propshaft

On multiple occasions, [`propshaft`](https://github.com/rails/propshaft) mentioned that it would become the default in Rails, so I chose it because I think it will make upgrading Rails easier. I also like that `propshaft` is simple and does one thing well.

### Beautiful web UX: Hotwire

Of course, [Hotwire](https://hotwired.dev) is my choice for building web UX. It comes with Rails by default, and it has a lot of good integrations. It allows me to build promising web UX without the mental overhead of learning more JS or a new JS library.

When I need to write some JS, I like to use Stimulus because it gives me a place and a way to organize JS code. This is important for a project you don’t open daily to know exactly where to find each piece of code.

### Environment variables: direnv

I started using [`direnv`](https://direnv.net) a while back, mostly because I needed to share some environment variables between projects. So I can put a `.envrc` file in the top folder for a group of projects and have those environment vars available in all subfolders.

It removes the need to use the `dotenv` gem and also decreases the chances of accidentally committing your `.env` file if you put the `.envrc` in the parent folder and not the code folder.

I mostly organize my projects like this:

```bash
projects/
project/shortruby/
project/shortruby/apps
project/shortruby/apps/shortruby.com
project/shortruby/apps/lean.shortruby.com
project/shortruby/apps/newsletter.shortruby.com
project/shortruby/images
project/shortruby/docs
project/shortruby/scripts
```

I can add the `.envrc` file under `projects/shortruby/apps` and have the common development environment variables shared between multiple projects.

### Code Quality: Rubocop + Rubycritic

I use Rubocop to enforce [a coding style](https://github.com/lucianghinda/personal-ruby-style) that I like and made for myself. It is well-supported by any editor and allows me to configure it as I see fit. In my case, I lean toward using new Ruby syntax so the cops are configured to allow that.

I added to it the following gems:

* [rubocop-rails](https://rubygems.org/gems/rubocop-rails) - “Automatic Rails code style checking tool”
    
* [rubocop-performance](https://rubygems.org/gems/rubocop-performance) - “A collection of RuboCop cops to check for performance optimizations in Ruby code”
    
* [rubocop-minitest](https://rubygems.org/gems/rubocop-minitest) - “Automatic Minitest code style checking tool”
    
* [rubocop-capybara](https://rubygems.org/gems/rubocop-capybara) - “Code style checking for Capybara test files (RSpec, Cucumber, Minitest)”
    
* [rubocop-rubycw](https://rubygems.org/gems/rubocop-rubycw) - “Integrate RuboCop and ruby -cw. You can get Ruby's warning as a RuboCop offense by rubocop-rubycw”
    

I also use [Rubycritic](https://github.com/whitesmith/rubycritic) to keep complexity low. I don’t refactor for every code smell, but it is good to have an excellent overview of what complexity each change might add to the existing codebase.

You can see how I configured all these tools in my article about [First commits](https://learn.shortruby.com/blog/first-commits).

### Code editor: RubyMine

I am using [`RubyMine`](https://www.jetbrains.com/ruby/) as my primary code editor with `IdeaVim` plugin enabled. I worked with it quite a lot in the last few years and am fast when using it. Helps with debugging and go to definition.

### Versioning: Github

I have some experience with both [Github](https://github.com) and [Gitlab](https://gitlab.com). But I think GitHub is a good choice here due to its UI simplicity. I don’t want to spend time in configuration and the UI/UX choices are great. The PR flow is quick and easy to set up, and for my needs, I only use GitHub actions to run simple tasks. I think Gitlab has a more powerful CI but I don’t think I need it right now.

### Sending emails: Postmark

For sending emails, I choose [Postmark](https://postmarkapp.com). It is easy to set up and offers API-sending email and SMTP services.

I don’t know if Postmark will be used to send the course content. But it will be used to send all transactional emails related to the project.

I am considering [ConverKit](https://convertkit.com) to write and send the actual email content. This will depend on their API support for the features I plan to build for my users: restart an email course, choose periodicity, and more.

### Hosting: Hetzner and Hatchbox

I am using VPS [Hetzner](https://www.hetzner.com) as they have a good price and being in EU it helps me with registering the invoice to my accounting.

I am using deployment [Hatchbox](https://hatchbox.io) for ease of use. I enabled automatic deployments to make a deploy with `git push` - simple and effective.

### Analytics: Plausible

I discovered [Plausible](https://plausible.io) while learning Elixir and Phoenix and started using it for all my side projects. I like the simplicity, and it is also a good price for what it is offering.

### Image storage: Cloudflare R2

I started looking for an AWS S3 alternative primarily due to the time spent configuring IAM and other stuff that always needed to be done when setting up a new account.

I discovered Cloudflare R2 last year (https://www.cloudflare.com/developer-platform/r2/), and I started using it. Again, I like the simplicity (it can be configured in minutes) and can be used for ActiveStorage. What I plan to build, where I will mostly upload and display images, worked perfectly and for an excellent price.

AWS S3 could be better for more complex tasks, but if the only need is to upload images and display them publicly while being a solo developer, the R2 is a good choice.

## Conclusion

These are the tools I use and why I chose them to build [the email courses project](https://learn.shortruby.com)

I don’t think there exists a thing called “the best tech stack,” but depending on my knowledge and purpose, I can create an excellent tech stack to help me deliver fast and with good code quality.