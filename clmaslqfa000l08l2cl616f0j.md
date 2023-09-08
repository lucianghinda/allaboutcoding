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

Next, I moved from the Edit with LLM functionality to Chat with LLM functionality that Cursor IDE offers.

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

What I notice in the response is that indeed [`h`](https://api.rubyonrails.org/classes/ERB/Util.html#method-c-h) is an alias for `html_escape` and that the proposed change makes sense.

What's important to notice is that when writing this prompt I already knew that something was wrong with the code and had an idea about what the fix will be.

Thus I asked a follow-up question:

```markdown
In this context is `html_escape` or `h` enough to mitigate 
the security risk of displaying an URL parameter provided 
by the user inside an ERB file?
```

Here is the response:

![GPT4 response about using html_escape](https://cdn.hashnode.com/res/hashnode/image/upload/v1694060388888/a14db7f7-37d9-4b8f-9a97-77452e1e822f.png align="center")

I was happy with the response and then moved on to finish my implementation.

You can see the PRs that I implemented with the cursor at:

* [Add seach component to all other pages](https://github.com/ShortRuby/rubyandrails.info/pull/105)
    
* [Refactor Search Term Display into a Reusable Component](https://github.com/ShortRuby/rubyandrails.info/pull/107/files)
    

## Some temporary conclusions

I used very simple prompts. Almost no context was given except for the ruby files or code itself. Nor did I ask for some proper follow-ups to nudge it in the desired direction. There was also no instruction about what a good code looks like for me.

I already had a good idea of what to look for thus, it was easy to know when a result was what I expected and when it was not

The changes that I made were small thus, it was easy to assess the code.

---

Enjoyed this article?

Join my [**Short Ruby News**](https://shortruby.com/) **newsletter** for weekly Ruby updates. For more Ruby **learning resources**, visit [**rubyandrails.info**](http://rubyandrails.info). You can also find me on [Ruby.social](https://ruby.social/@lucian) or [Linkedin](https://linkedin.com/in/lucianghinda)