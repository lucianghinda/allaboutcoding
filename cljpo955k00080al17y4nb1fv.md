---
title: "A comparison of multiple generative AI tools when asking for Ruby on Rails code"
datePublished: Wed Jul 05 2023 12:05:02 GMT+0000 (Coordinated Universal Time)
cuid: cljpo955k00080al17y4nb1fv
slug: a-comparison-of-multiple-generative-ai-tools-when-asking-for-ruby-on-rails-code
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1688558607458/f7fe440f-acb1-4855-b3c4-92aae90ae3a7.png
tags: ai, ruby, ruby-on-rails, ai-tools, generative-ai

---

I asked the following query for multiple AI code generator tools:

````markdown
Please transform this code:
```
<div class="flex justify-center py-6 mt-8 mb-3" role="separator">
  <span class="dot"></span>
  <span class="dot"></span>
  <span class="dot"></span>
</div>
```
into a Rails Helper with content tags
````

I tried to make the request a mix of simple wording while also providing a small context by specifying the `content tags` and mentioning `Rails Helper`.

## Chat GPT

Here is the [ChatGPT](https://chat.openai.com) response:

![ChatGPT response](https://cdn.hashnode.com/res/hashnode/image/upload/v1688530493250/ad93ac4a-78e2-404c-81d9-2b83c306da5d.png align="center")

It works, it will display the HTML as required.

My main review here is that I would use `concat` instead of `join.html_safe`

So I did a follow-up:

![Followup with ChatGPT asking to remove html_safe](https://cdn.hashnode.com/res/hashnode/image/upload/v1688531042782/27e44625-a05c-4631-9797-454aaec44960.png align="center")

Of course, this was not what I was looking for, but I also agree that my request was not very specific. I did this intentionally to keep the conversation casual, and not suggest a specific solution. I also notice that it changed the second param of `content_tag` from an empty string to `nil`. I did not request this specific change but I assume asking `refactor` triggered a wider change.

I again follow up with the request to remove `safe_join`:

![Follow up with ChatGPT asking to remove the safe_join](https://cdn.hashnode.com/res/hashnode/image/upload/v1688531139711/98f0ca85-871d-46af-8aa2-cb514030e09f.png align="center")

And this indeed is the solution that I think is a bit close to what I wanted. I don't think `.map` is needed there and I usually like helpers to make things explicit so I would have written 3 lines of `concat(content_tag)` but I think in general the solution is usable.

## Phind

[Phind](https://www.phind.com) describes themselves as *"The AI search engine for developers"*.

Here is the response from Phind:

![Response from Phind showing a DivHelper that has a method dot_separator](https://cdn.hashnode.com/res/hashnode/image/upload/v1688531368588/9fda455a-8964-4258-98a6-5d2ddc7a4bae.png align="center")

I was quite happy with the result:

* Good name `dot_separator`
    
* I like the idea that it extracted the actual dot to a private method `dot_span` but I think the name could have been improved to only `dot`.
    
* I like that it used `concat`
    
* And I welcome the explicitness of having 3 lines of `contact(dot_span)` instead of `3.times { contact(dot_span) }`
    

I did not feel here that I need to follow up so I moved to the next tool.

## Cody

[Cody](https://docs.sourcegraph.com/cody) is an AI assistant from Source Graph - *"Cody is an AI coding assistant that writes code and answers questions for you by reading your entire codebase and the code graph"*

Here is the first response from Cody:

![First response from Cody AI](https://cdn.hashnode.com/res/hashnode/image/upload/v1688532008501/ef7380c3-6eb0-4e06-a625-e5861bad1d20.png align="center")

This will not work as intended as it will only display one single dot.

So I said to Cody:

![Followup with Cody AI describing that it does not work as it shows one single dot](https://cdn.hashnode.com/res/hashnode/image/upload/v1688532070321/21614697-fa59-4c49-b7e0-d24f7dd00039.png align="center")

Now this code will not display anything, so I asked Cody to fix it:

![Second followup with Cody AI asking to fix the bug where it does not show anything](https://cdn.hashnode.com/res/hashnode/image/upload/v1688532158833/f168bb1c-cf92-4787-8bc0-eede273c5277.png align="center")

Again this code will not show anything so I asked again Cody to fix it but now I asked also to explain why it does not work. This is a trick with LLMs that will usually give better results because it brings more context about what is not working and the cause of it:

![Third follow up with Cody AI saying again it does not work and asking to fix it](https://cdn.hashnode.com/res/hashnode/image/upload/v1688532242727/e5c28101-b2a0-4ce4-8af1-e94a8d9b7af8.png align="center")

Now this works - it will display 3 dots, but it is still a version that I don't like as it uses `html_safe`. So I asked for a refactor:

![Asking Cody AI to refactor to remove usage of html_safe](https://cdn.hashnode.com/res/hashnode/image/upload/v1688532408929/e6e97f7d-7381-4d9f-ae4e-d257ae0c03dc.png align="center")

This works and it is a solution that I consider ok(ish).

## Github Copilot Chat

[**Github Copilot Chat**](https://github.com/github-copilot/chat_waitlist_signup) is *"A ChatGPT-like experience in your editor with GitHub Copilot chat"* that I installed in a VScode Insiders version.

Here is the response I got from it:

![Github Copilot Chat response](https://cdn.hashnode.com/res/hashnode/image/upload/v1688532643905/980c2492-5c6b-4324-9f2f-82821ccc2114.png align="center")

First, this solution works. Then I like that it tried to give a good name `separator_dots` and I appreciate that it used `concat` instead of first trying to use `html_safe`

Copilot Chat is a version of the Open AI GPT-4 model I think but probably has a better system prompt.

## Rix

[Rix](https://hashnode.com/rix) is a Hashnode tool described as *"The AI search companion, optimized for developers"*

Here is the response I got:

![Rix response](https://cdn.hashnode.com/res/hashnode/image/upload/v1688533066934/ceb2e7d6-774f-45a8-8b91-9ee82c179a43.png align="center")

## (temporary) conclusion

In the context of Ruby programming, AI code generator tools offer varying results and sometimes require follow-ups to achieve the desired output. While some tools, like Phind and Github Copilot Chat, provide satisfactory solutions quicker, others may need additional guidance. It is essential to thoroughly review the generated code to ensure security and avoid unintended changes.

## Word of caution

**Do not feed private information:** I used this with a piece of code that does not show any private information about the project I worked with.

**Be careful what code you feed into these tools:** When using generative AI tools - I only add those to my side projects and most of the time when I know that I will anyhow make the project open source. I would not use them or give them access to a client code base unless explicitly accepted by the client in a contract.

**Be careful of security issues:** Also please be careful of copying/pasting code generated by AI in your project and running that code on servers. Nobody can guarantee that the code does not have security issues.

**Read all the generated code, not just the changed parts:** One last point, if you are asking for refactoring, read every time the entire code and not only the lines that you required to be changed. Sometimes the AI will change some other things that you did not specifically ask for.