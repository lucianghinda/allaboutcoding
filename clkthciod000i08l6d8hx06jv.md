---
title: "Projects ideas for learning Ruby or any Ruby web framework"
seoTitle: "Projects to code while learning Ruby and Ruby on Rails"
seoDescription: "Discover beginner Ruby & Rails projects: Meal Planner, Lists Share, Habit & Symptom Trackers, SWOT Analysis; learn by problem-solving"
datePublished: Wed Aug 02 2023 08:42:29 GMT+0000 (Coordinated Universal Time)
cuid: clkthciod000i08l6d8hx06jv
slug: projects-ideas-for-learning-ruby-or-any-ruby-web-framework
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1690965437292/f42d0c24-14fe-4282-8299-3a60cc1926c0.png
tags: ruby-on-rails, learning, reactjs

---

*I shared this on* [*Twitter*](https://twitter.com/lucianghinda/status/1354069119704977409) *a while back (you can* [*read it also on nitter*](https://nitter.net/lucianghinda/status/1354069119704977409)*) and I think it is better to be published on the web instead of just remaining there of course also adding some more context to it.*

When I talk with someone that is learning Ruby or Ruby on Rails I usually receive this question: what project should I build when I learn Ruby on Rails?

The same goes for me: when I am learning something new, I feel the need to build something with it. And it works best if what I am trying to build is as real as possible and not only a "theoretical" abstract learning project.

In this article, I will try to make the case that it is best to:

1. Try to solve a problem you have while learning a new programming language or framework
    
2. If that does not work or if it feels too hard to tackle in the beginning I will present a list of projects that I think are good to be picked up as a learning project
    

## Solve a problem/need you have

I think from the learning process point of view the best project to work on is to solve a need you have. This works best because it will keep your motivation up and you will also get enthusiastic about seeing that using Ruby will help you directly.

### **Create a script to solve your problem**

I recommend starting with this more if you are learning Ruby now and did not yet start to learn a web framework.

A script that will take some inputs, execute some logic and then output something on the terminal is a small time investment and will let you focus on Ruby and writing idiomatic code instead of dealing with UI.

I have a lot of scripts written in Ruby. Here are some examples:

* parse CSVs and extract information from there: you can do counts, averages, sum ... on some specific fields. I use this for example to parse Stripe exports and some other financial exports and then get quick information like total amount per month, unpaid invoices ...
    
* a script that takes a screenshot of a web page and will save it in a specific folder
    
* a script that will read the `<title>` and `<meta name="description" content="...">` and will print them out
    

I can go on with more examples, but I hope you got the idea. Write a small script that is executing some code that will solve one single need.

How to approach this:

1. Write a script that will solve your problem and make it as specific as possible. Eg if you are writing a parse CSV script to read an export you have and print a total sum, just write this script based on the CSV file that you currently have. Don't try to make it work with any CSV or try to make it work with columns being moved around or renamed.
    
2. Then try to redesign the code and improve the code quality
    
3. Then try to make your script more generic or customizable: like accepting a different CSV file with other columns and other column names.
    

I know this probably sounds very simple, but trust me. I got so many times into rabbit holes trying to write a script that can handle so many edge cases and be so flexible and I almost every time invented a kind of framework.

### **Create a simple web app to solve your problem**

Again my recommendation is to look at a problem or need that you have and try to build a simple web app for it.

The same advice applies:

* Just focus on directly solving the problem. Please don't add authentication to it from the beginning or automated deployment or any extra flexibility. Pick one single thing and focus on solving that.
    
* Make it work on your computer. Don't bother to create a big architecture that will handle millions of edge cases. That is good learning maybe, but not the first time you learn a web framework.
    
* Try to focus on following the guides for that specific web framework. Even if you disagree with some of the things there, it is good to understand first what are the cases that the framework solves and how it tries to solve them before going into making it work for you.
    
* Then after you have something working you can now expand your web app to handle more cases.
    

If you want to pickup some good habits when building a web app, after you did the initial version consider working to improve the following 3 areas:

1. Accessibility
    
2. Security
    
3. Performance
    

## Examples of web apps that you can build

The following is a list of possible web apps that you can build as a learning process. I choose them because I think they are easy to be approached and their domain knowledge is not hard to understand.

### Weekly Meal Planner

A web app where users will submit their recipes, food items and then do a meal plan for a week (breakfast, lunch, dinner). Bonus if you can support multiple family members and allow them to vote on which day they liked the most.

Here is a possible list of features and the order to build them:

1. Create a new recipe with a title, description, steps, and time to cook
    
2. Create a Week with an attribute called Title
    
3. Add multiple recipes to a Week
    

From here you can add accounts, owners, voting ...

### Lists Share

An app that allows a user to create a list of things gives a short tile of that list and a description about it, and then after saving it allows the user to share it with anyone. A visitor should be able to see the list without being authenticated.

Here is a possible list of features and the order to build them:

1. Create a list with a title, description
    
2. Add items to a list
    
3. Create users
    

From here you can add more complex features:

* Display a view counter
    
* Add a feature to allow authenticated users to do a “thumbs up” kind of vote for other users' lists
    
* display on your app's home page the top 10 most-seen lists and the last 10 recently created lists
    

### Habits Tracker

Create a web app that will allow the user to set up a list of habits the user wants to track daily and then ask every day user to mark with a check if they made that habit or now that day. Display completion progress for 30 days.

Here is a possible list of features and the order to build them:

1. Create a habit
    
2. Mark a habit as done for today
    
3. Display a calendar view showing if the habit was completed each day
    

From here you can add more complex features:

* Tracking multiple habits per day
    
* Add users
    
* Display a graphic about how the completion of each habit worked (see D3 or Charkick JS libraries for inspiration - both have rails gems)
    
* Send a monthly email with a summary of progress for each habit.
    

### Symptoms Tracker

An app to track symptoms daily. The user should add one or more symptoms by clicking on a specific day and choosing the time and adding a text describing the symptoms. Display a monthly calendar and mark the days with symptoms.

Here is a possible list of features and the order to build them:

1. Add a symptom for today from a predefined list
    
2. Add a custom symptom for the current day
    
3. Add symptoms in the past: choose any day and add one or many symptoms
    

From here you can add more complex features like:

1. Add a tag for the symptoms and categorize them
    
2. Display the evolution of a tag/symptom during a week/month. Like adding a scale for each symptom
    
3. Add users support
    
4. Allow for a user to track symptoms for multiple people (like a partner, child …)
    

### SWOT Analysis

Create a web app to allow users to create a SWOT analysis on any topic. This should allow the user to fill in all 4 quadrants and then share the Swot Analysis with other people.

Here is a possible list of features and the order to build them:

1. Create a new SWOT analysis and fill in the 4 quadrants
    
2. Create a unique shareable link for each SWOT analysis
    
3. Edit SWOT analysis
    

From here you can add more complex features like:

1. It should also allow adding comments to each section of the analysis.
    
2. Add users
    
3. Allow other users to suggest new lines to be added to each section and let the owner accept it or reject it
    

---

Enjoyed this article?

Join my [**Short Ruby News**](https://shortruby.com/) **newsletter** for weekly Ruby updates. Also, check out my co-authored **book**, [**LintingRuby**](https://lintingruby.com/), for insights on automated code checks. For more Ruby **learning resources**, visit [**rubyandrails.info**](http://rubyandrails.info)