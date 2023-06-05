---
title: "This week focus: Documentation"
datePublished: Mon Jun 05 2023 08:04:15 GMT+0000 (Coordinated Universal Time)
cuid: cliikfycq000f09ms8zqa7rt2
slug: this-week-focus-documentation
tags: ruby, rails, documentation, focus

---

Whenever you open a piece of code try to find a place where you can describe how that class or that feature can be used by fellow developers.

Focus on usage examples or use cases.

**You can start small with a PR description.**

Add some examples of usage in the PR description. This will be an easy start without any pressure, and it can be edited as required based on feedback from your colleagues.

**Add examples using YARD**

Another way is to use YARD structure to write before a class or method directly:

```ruby
# @example How to make this use case work
#    ```
#    t = http://MyObect.new(...)
#    t.run(email_param: "test")
#    ```
```

Try to write an example that, when copied/pasted to `irb` or `rails console` will work.

I like this because most IDEs know how to show YARD examples while writing the code and thus you will have examples of usage for your classes while typing.

**Write documentation in markdown format**

If you don't like to write documentation inside your code files, put your documentation outside your code files.

The easiest is to create a `./docs` folder inside your project and write in a markdown file. Both Github and Gitlab already know how to render markdown files thus you will have browsable and searchable documentation without any extra effort.

In case you want something more fancy [mdBook](https://rust-lang.github.io/mdBook/index.html) looks very good for providing a way to display this documentation.

### The best is to start anywhere

There are maybe other ways to write documentation. Focus this week on this. You will thank yourself in a couple of weeks if you will need to change the code you wrote today if you wrote documentation about it somewhere.

---

Enjoyed this article?

Join my [**Short Ruby News**](https://shortruby.com) **newsletter** for weekly Ruby updates. Also, check out my co-authored **book**, [**LintingRuby**](https://lintingruby.com), for insights on automated code checks. For more Ruby **learning resources**, visit [**rubyandrails.info**](https://rubyandrails.info).