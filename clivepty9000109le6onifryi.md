---
title: "Why write technical content on a blog and not only on social media"
seoTitle: "Publishing your content on a blog content vs. sharing on social media"
seoDescription: "Use a personal blog for technical content to ensure resilience, avoid platform dependency, and prevent account suspension"
datePublished: Wed Jun 14 2023 07:44:59 GMT+0000 (Coordinated Universal Time)
cuid: clivepty9000109le6onifryi
slug: why-write-technical-content-on-a-blog-and-not-only-on-social-media
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1686628581492/150bf307-9da1-4435-8ae0-e984d8dda588.png
tags: blogging, ruby, share, content-creation

---

I often recommend that people publish their social media content as a blog post or a GitHub gist if they don't have a blog.

Here are the reasons:

* **Content Resilience**: No matter what happens on the social media platform, your content will remain available for everybody. Take, for example, the migration that happened with some users who left Twitter and closed their accounts. Their content is no longer easily accessible. Maybe it was archived by the Internet Archive, or maybe not.
    
* **Don't depend on a platform to reach your audience**: Limiting your content sharing to any (or more) social media platform makes you dependent on it. If any issues arise with your account, such as being blocked, you will lose the ability to communicate with your audience.
    
* **Safeguard against unjust or erroneous termination or suspension of your social media account:** Picture the worst-case scenario where an incident occurs and your account gets suspended. Suddenly, your content and, consequently, your voice will no longer be heard.
    

These reasons also apply to any website where you use a subdomain (eg: username.example.com) or path (eg: example.com/username) rather than your own domain.

## What to do

1. Get a domain
    
2. Create a blog with your domain or find a platform that offers this feature
    
3. Ensure your blog has RSS support
    
4. When you write quality content on social media, also post it on your blog. You can do this before or after sharing on social media.
    
5. Make sure you have a backup setup for your own content
    
6. (optional) Add newsletter support so people can get your content in their email when you post something.
    

Optional:

* I recommend you choose a blogging engine/platform where you can write your content in markdown format or at least they can export the content in markdown. This will ensure you can easily migrate between blog engines without any effort to rewrite your content
    
* I also recommend choosing a blogging engine that will help you with some kind of automatic backup. Here I like to set up a flow that will create a commit with the markdown files in Github.
    

That's it. Now your content is safeguarded from any dependency on a specific platform.

## Simple recommendations

Regarding the selection of a blog engine and its setup, I will share my recommendations here. However, please note that these suggestions may not necessarily suit everyone's needs.

### Build everything yourself

If you want to have a different UI or your blog to look in a very specific way I recommend using [Jekyll](https://jekyllrb.com) or [Bridgetown](https://www.bridgetownrb.com).

With both options, you can directly host your blog on GitHub. See documentation [here for Bridgetown](https://www.bridgetownrb.com/docs/bundled-configurations#github-pages-configuration) and documentation [here for Jekyll](https://pages.github.com).

### Use an existing blogging platform

I am using it for this blog [Hashnode](https://hashnode.com) because they offer two options that I like a lot:

* custom domains so I can set up my domain for this blog
    
* save to Github - Hashnode offers the option to back up what you write to Github and also publish from Github. [Checkout this option](https://townhall.hashnode.com/start-using-github-to-publish-articles-on-hashnode). This way everything that I publish here is saved on [my Github repo](https://github.com/lucianghinda/allaboutcoding).
    

They also recently added newsletter support and sponsorship.

If you are the kind of person (like me) that will spend days customizing their theme and then configure servers and then start building new features for a blogging platform I suggest you choose an existing blogging platform and just focus on writing.

---

Enjoyed this article?

Join my [**Short Ruby News**](https://shortruby.com/) **newsletter** for weekly Ruby updates. Also, check out my co-authored **book**, [**LintingRuby**](https://lintingruby.com/), for insights on automated code checks. For more Ruby **learning resources**, visit [**rubyandrails.info**](http://rubyandrails.info)**.**