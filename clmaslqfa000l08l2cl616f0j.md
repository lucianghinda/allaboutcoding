---
title: "Using Cursor IDE for some small changes in a Rails app"
seoTitle: "How to use Cursor IDE to make changes in a Rails app"
seoDescription: "How to use Cursor IDE to generate for Ruby on Rails apps: examples of prompts, responses and improvements"
datePublished: Fri Sep 08 2023 16:09:22 GMT+0000 (Coordinated Universal Time)
cuid: clmaslqfa000l08l2cl616f0j
slug: using-cursor-ide-for-some-small-changes-in-a-rails-app
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1694189086850/959a2e87-c56b-495a-affa-85f7b75c42f6.png
tags: ai, ruby, rails, ides, cursor-ide

---

Path of my learning path about how to use AI/LLMs to augment my developer productivity I started using [www.cursor.so](https://www.cursor.so)

Here let me show you how I made two changes to the [rubyandrails.info](https://rubyandrails.info) website.

## Replacing a form with a Phlex component

The first task that I asked was about replacing an HTML with a component. To achieve this in Cursor (MacOS edition) you have to select the text and then press CMD+K. Then a pop-up will appear where you can ask your question (or write your prompt) like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694187809116/87d1c036-b168-45a0-add3-5348446119f4.png align="center")

After submitting Cursor (that uses GPT4 in this case under the hood) will make a diff for the selected code and ask you to accept it or not:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694187930106/e567c53a-7bd2-4763-a3c5-63c362b86a8b.png align="center")

I like this idea that it proposed a diff because I can review it and check if it is correct or not.

This was a small change and I was happy with the result. I did a lot more replaces like this but I did not use Cursor LLM for this.

**My conclusion** is that for this very small case, writing the prompt and reviewing the change is much more effort than directly writing the code myself and the return on investment is small. It does not even contain any new insights or something to learn for me.

What I would like (and maybe Cursor knows this but I am just starting to use it) is to ask to replace all forms like that with the component. I will still need to review the code because the risk would be to replace a code that looks like that but it is not the same search functionality.

## A small security PR

Next, I moved from the Edit with LLM functionality to the Chat with LLM functionality that Cursor IDE offers.

Looking at the source code I noticed the following code, which I think it is a security risk.

```erb
<strong>Search Term: </strong><%= params[:search_term] %>
```

Thus I asked the following to the Cursor chat:

![GPT4 prompt to ask for security analysis](https://cdn.hashnode.com/res/hashnode/image/upload/v1694057596283/9774c46e-8518-4680-b073-ea7746c722b4.png align="center")

One advantage I found while using Cursor chat is that it makes it easy to reference open files in a prompt.

Notice that I opened `_index_nav.html.erb` and reference it in my prompt with `@_index_nav.html.erb` and Cursor read the content to provide it to GPT4. The same can be achieved by selecting the text and pressing `CMD+L` will add the code itself as the context in the chat. By the way with CMD+L you can add multiple pieces of code from multiple files (but we will explore that in another article).

Here is the response:

![Response from Cursor chat](https://cdn.hashnode.com/res/hashnode/image/upload/v1694057733418/5afc1558-facf-4d22-a9f9-3e4ed2f90b7c.png align="center")

What I notice in the response is that indeed [`h`](https://api.rubyonrails.org/classes/ERB/Util.html#method-c-h) is an alias for `html_escape` but the response is a bit outdated. Since [Rails 3.0](https://guides.rubyonrails.org/3_0_release_notes.html#other-changes) there is no need to escape with `h` because:

> * You no longer need to call `h(string)` to escape HTML output, it is on by default in all view templates. If you want the unescaped string, call `raw(string)`.
>     

The response, although it won't break anything, is also unnecessary and moreso it is incorrect. It is incorrect because until a vulnerability - if any - is found in the escape Rails does, the code should be considered safe. If we don't do this then we cannot work with abstractions or trust a framework.

Furthermore, since this information is from Rails 3.0, GPT-4 should already be aware of it.

I followed up with this question:

```markdown
In this context is `html_escape` or `h` enough to mitigate 
the security risk of displaying an URL parameter provided 
by the user inside an ERB file?
```

Here is the response:

![GPT4 response about using html_escape](https://cdn.hashnode.com/res/hashnode/image/upload/v1694060388888/a14db7f7-37d9-4b8f-9a97-77452e1e822f.png align="center")

Also, the response here is a bit out of place. We are in the context of an ERB file where the code is just printing a variable. There is no SQL query executed. Again while the response sounds logical apparently it is unhelpful. It also shows that once the GPT gets into a direction it hardly self-corrects itself unless instructed to do so.

I still decided to extract into a component the display of the search and thus have it ready for further UI improvements across all pages.

You can see the PRs that I implemented with the Cursor IDE at:

* [Add search component to all other pages](https://github.com/ShortRuby/rubyandrails.info/pull/105)
    
* [Refactor Search Term Display into a Reusable Component](https://github.com/ShortRuby/rubyandrails.info/pull/107/files)
    

## Some temporary conclusions

I used very simple prompts. Almost no context was given except for the ruby files or code itself. Nor did I ask for some proper follow-ups to nudge it in the desired direction. There was also no instruction about what a good code looks like for me.

I already had a good idea of what to look for. Thus, it was easy to know when a result was what I expected and when it was not.

The changes I made were small. Thus, it was easy to assess the code.

One challenging task is to determine if a response is up-to-date, as demonstrated by the interaction about escaping the parameter.

Another challenging task is to be able to detect a response that sounds logical, but is out of context and thus would be wrong to be applied in that specific situation.

## Update

1. In a previous version of this article, I was evaluating the response from GPT-4 which recommends using `h` to escape the HTML output when using in ERB as good enough. [Xavier Noria](https://hashref.com) pointed out that it is not needed to use `h` as Rails ERB `<%=` is escaping HTML output by default. I confirmed this by testing and also found the [changelog that mentions this since Rails 3.0](https://guides.rubyonrails.org/3_0_release_notes.html#other-changes).
    
2. After a second review from [Xavier](https://hashref.com) and following a discussion we had I am reconsidering my evaluation of the response about XSS vulnerability from unnecessary to incorrect. Until proven otherwise (by someone else finding a bug in the html\_escape) the escape provided by Rails should be considered safe and this should be the correct response from LLM.
    

---

Enjoyed this article?

Join my [**Short Ruby News**](https://shortruby.com/) **newsletter** for weekly Ruby updates. For more Ruby **learning resources**, visit [**rubyandrails.info**](http://rubyandrails.info). You can also find me on [Ruby.social](https://ruby.social/@lucian) or [Linkedin](https://linkedin.com/in/lucianghinda)