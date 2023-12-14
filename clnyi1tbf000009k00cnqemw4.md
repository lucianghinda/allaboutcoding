---
title: "Using ChatGPT, Github Copilot and Phind to generate Tailwind config for width classes"
seoTitle: "ChatGPT, GitHub Copilot: How to generate Tailwind width configuration"
seoDescription: "Compare ChatGPT & Github Copilot's methods for creating custom Tailwind width classes with golden ratio; evaluate effectiveness and accuracy"
datePublished: Fri Oct 20 2023 11:00:07 GMT+0000 (Coordinated Universal Time)
cuid: clnyi1tbf000009k00cnqemw4
slug: using-chatgpt-github-copilot-and-phind-to-generate-tailwind-config-for-width-classes
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1697801455326/e065a4e3-a08d-4097-9299-8f4b89de16e5.png
tags: ai, ruby, ruby-on-rails, tailwind-css, chatgpt

---

In this article, I will show how I used ChatGPT, Github Copilot and Phind to generate a custom Tailwind config for width classes up to 800px for desktop resolution.

These language models can serve as helpful conversation partners, providing different solutions to the same problem.

Here, we'll compare their responses to a specific request involving the golden ratio and discuss their effectiveness.

## Problem

As this was a personal project, I did not have a complete design, just some inspirational ideas that I wanted to try.

I wanted to add some custom width-sized in a Tailwind config for a web app up until about 800px for the desktop resolution.

I asked both Chat GPT 4 and Github Copilot Chat the same question and got different answers.

## Chat GPT 4

Here is what I asked Chat GPT 4:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697726678692/92c6083a-5624-458c-81c2-9bbe8c750f70.png align="center")

Here is what it suggested:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697726704852/adf0a679-7f42-4889-a216-7b01c26e8181.png align="center")

As you can notice it used the constraint that I added (to use the golden ratio) to calculate the actual size from the last one provided by me (384px) to reach the 800px that I asked for and then tried to create a series of width utilities matching a bit how Tailwind does it - by spacing them out.

Did not check all of the utilities proposed, but the first one and the last one are calculated ok from the rem to px ratio.

## Github Copilot

I then tried to use Github Copilot via VSCode to generate this list.

Here is the prompt and the response:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697729803377/e86239e8-0d0c-4f4c-bdbe-50c94230ae4a.png align="center")

It has some fine-grained steps, it reaches the 800 px and it correctly started from 400 (as Tailwind already defined 384px) but I don't see how it used the golden ratio (which is around 1.61).

It ignored the request to use the golden ratio: did not use it for spacing the utilities, nor used it for calculating the distance in rem or pixels between two utility classes.

## Github Copilot Chat

I then tried Github Copilot Chat:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697734445999/c7e90baf-7c0b-4d1b-82fd-65b38659b935.png align="center")

And it replied with the following:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697734476571/133df7c4-652b-40d4-8c12-f6d35166614d.png align="center")

It seems that it was not aware that Tailwind already has width classes up until 384px. But it did apply with an approximation of the golden ratio when spacing the width classes.

## Phind

I asked [Phind](https://www.phind.com) the same question and here is the answer:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697801297781/4c884fb9-62bc-4655-87b5-673d1eccccc0.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697801327316/722b2e88-d14f-42e6-bff8-7e728a9884f3.png align="center")

I find the Phind answer better. It explains how it did the calculation, it used the golden ratio for each step and it also provides the Tailwind config and shows how to use it.

---

Enjoyed this article?

Join my [**Short Ruby News**](https://shortruby.com/) **newsletter** for weekly Ruby updates from the community. For more Ruby **learning resources**, visit [**rubyandrails.info**](http://rubyandrails.info). You can also find me on [**Ruby.social**](https://ruby.social/@lucian) or [**Linkedin**](https://linkedin.com/in/lucianghinda) or [**Twitter**](https://x.com/lucianghinda) where I post mostly about Ruby and Rails. Subscribe to [**my YouTube channel**](https://www.youtube.com/@shortruby) for short videos about Ruby.