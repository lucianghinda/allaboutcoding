---
title: "How to learn Ruby"
datePublished: Fri Sep 02 2022 07:03:33 GMT+0000 (Coordinated Universal Time)
cuid: cl7k4qs150en84xnv5f9kdfzm
slug: how-to-learn-ruby
tags: ruby, learning, coding, programming-tips

---

I think learning Ruby has three parts:
1. OOP + SOLID in Ruby
2. Ruby Syntax
3. Idiomatic Ruby

What follows assumes you already know how to program in any other programming language. If this is the first time you are learning to code, then that is another path and I will write about it some day. 

I would start in the following way: 
1. Ruby Syntax: First, read a bit about the Ruby syntax. You can find a short introduction to Ruby. (shameless plug) I tried to collect a lot of resources [here](https://ghinda.com/blog/programming/ruby/2021/learning-ruby.html#learning-ruby-where-and-how), but you can also find some resources [here](https://rubyandrails.info). Do some small exercises if you feel like it just to get the syntax. 
2. Watch some videos of Sandi Metz. Here is a [list of Sandi Metz videos](https://www.youtube.com/results?search_query=sandi+metz) for that. I recommend starting with [SOLID Object-Oriented Design by Sandi Metz](https://www.youtube.com/watch?v=v-2yFMzxqwU). Some things might make sense, some not. No worries, this is just a kind of priming your way of thinking about how to code in Ruby
3. After watching one video go ahead and try to work on some learning project in Ruby. Do the first version of that that should work. Here is an example: Do an ATM simulator in Ruby. Or a Daily Quote CLI that should serve one quote daily. 
4. Push that project to Github and Gitlab. 
5. After you do the first version, install Minitest and try to redo that project but TDD style. So first, you will write a test and then make that test pass. Do this on a separate branch and open an MR/PR with it. 
6. Now, after you rewrote that project with tests, go ahead and rewatch the SOLID Object Oriented
7. After watching, go open Github/Gitlab and review your code and try to add comments on what you think should be changed. That PR/MR done previously will help with this. 
8. Here is how to do the review: First, think about one of the principles you saw explained in the video and read your PR/MR, and add comments there with what to refactor to follow that principle. Then think about the second principle explained and then reread the code and add comments about this second principle. This is called perspective-based review, and it works better when you focus on one thing at a time.
9. When finished with your review, go ahead and refactor based on the comments you added there.
10. Watch this video [Rules, by Sandi Metz](https://www.youtube.com/watch?v=npOGOmkxuio) and then go ahead and do the same process: Do the review and then refactor your code again.
11. Then watch this video [Aloha Ruby Conf 2012 Refactoring from Good to Great by Ben Orenstein](https://www.youtube.com/watch?v=DC-pQPq0acs) and again do the same process: review + refactoring

At this point, you will know Ruby Syntax and OOP principles in Ruby. 

---

It is time to go deep into idiomatic Ruby: 
1. I recommend first reading this https://www.poodr.com/
2. Then I recommend you to read this https://www.packtpub.com/product/polished-ruby-programming/9781801072724

You don't need to read those books from start to finish. I recommend reading a chapter and doing a code review, and then refactoring.

Then you can read some style guides, but you can search and read more of them: 
1. [Shopify Style Guide](https://ruby-style-guide.shopify.dev/)
2. [Thoughtbot Style Guide](https://github.com/thoughtbot/guides/tree/main/ruby)
3. [AirBnB Style Guide](https://airbnb.io/projects/ruby/)
4. [Cookpad Style Guide](https://github.com/cookpad/styleguide/blob/master/ruby.en.md)

See which style guide makes sense for you now, and then go ahead and refactor your code to match one of them. Do not try now to mix them, but just take one and refactor your code to follow the guide. 

After doing this, it is time that you start using static analysis tools:
1. Install [Rubocop](https://github.com/rubocop/rubocop) and see if any of the style guides that you read has a Rubocop configuration. Use it with Rubocop. 
2. Install [Ruby Critic](https://github.com/whitesmith/rubycritic) and run it and try to fix reported issues. 

From now you can start experimenting freely with Ruby and see what you like, what your style is what makes you happy. 

---
Follow me on Twitter [@lucianghinda](https://twitter.com/lucianghinda) where I share/retweet mostly Ruby and Rails content. 

I also publish a newsletter with Ruby and Rails fresh content at https://newsletter.shortruby.com - in case you want to stay up to date with what is happening in Ruby and Rails world. 

