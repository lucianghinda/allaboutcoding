---
title: "This week focus: Refactoring"
datePublished: Tue Jun 13 2023 08:04:18 GMT+0000 (Coordinated Universal Time)
cuid: clitzytcf000909mo64aw8xwg
slug: this-week-focus-refactoring
tags: refactoring, programming-blogs, ruby, ruby-on-rails, programming-tips

---

Look at your code and identify any areas where you can improve. Refactoring isn't about fixing bugs but improving your code's readability, efficiency, and maintainability.

Look for complex methods that could be broken down, repeated code that could be extracted into its method or class, and any design patterns that could be better applied.

But mostly, ask yourself: How much effort will it take a new colleague to understand what this code does?

For inspiration, take a look at these eloquent words from Yukihiro Matz (from Beautiful Code book). I get back to reading that chapter from time to time when I contemplate refactoring and how I write Ruby code:

![A quote from Yukihiro Matz about beautiful code](https://cdn.hashnode.com/res/hashnode/image/upload/v1686636650247/4a7273fd-a96f-466f-83e2-e6953b9450ad.png align="center")

## Start small

1\. Open a file

1. Identify a method that you think is complex or hard to understand
    
2. Make sure you have a test that covers functionally what that piece of code should do (could also be indirect)
    
3. Choose one thing that you want to improve: make it smaller or readable or better naming ...
    
4. Refactor it
    
5. Ask for a review with at least a focus on readability
    

Of course if you have more time then there are other strategies to approach refactoring. But in case you don't have too much dedicated time, let's focus this week in doing some bit and pieces of refactoring.