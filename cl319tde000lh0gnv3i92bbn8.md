## Why participate in a coding competition

Imagine you are taking part in a coding competition. Let’s explore what it might look like.

You found this contest online and registered to participate. On the day of the event, you confirm your presence and, after finishing checking in, you unpack your laptop and find a comfy place to sit.

You watch the other people in the room before your competition starts. Most of them are focused on their laptops, making last-minute adjustments to sitting positions, to their programming environments, and preparing mentally for what is about to come. Some of them are in small groups, chatting or even laughing to wash away possible nervousness.

The problems are finally published and you dive into solving them.

Time passes by and you go back and forth through a series of experiences. There are moments when you feel confident that you’ve found the best solution. There are also moments of uncertainty, discovery, thinking, and creating strategies. Time passes unaffected by what is happening in the room.

The big finale is approaching very quickly. You look at the clock and then again at your computer screen. With only a few minutes left, you are not sure if the code you wrote will be enough to win this contest.

Also, you are now facing a hard choice: **(a)** should you rewrite part of it, but maybe not have time to completely test the code, or **(b)** just continue with the current code and make small, incremental adjustments, while being sure it runs and works? 
The choice is hard also because you don’t know what your opponents did, or what they implemented.

---

The description above might be what you could experience during a programming contest. Or you might experience many other different things, as not all experiences are the same.

There are a lot of reasons to participate in this kind of event, no matter the experience you have so far or the way you will behave during these events

---

## Learning to write code faster

Competition is limited in time and usually, there is no way to extend the allowed interval. You are not just solving problems, you also need to do it quickly. Writing code faster will allow you to go through many possible solutions and pick the best one out of them.

A big part of writing code fast is to intimately know your IDE, especially the shortcuts it provides. Other things that improve coding speed may be having experience with library data structures and methods, using standard best practices when choosing variable names and method structure, structuring code on paper before writing it, etc.


## Focus on making your solution work

Yes, during a competition you could also focus on writing clear, beautiful code. But the first objective you need to handle is making your code to work. Implement your solution in the fastest way possible so that it works. Only after it works, you could then start to improve it, making it look good and clean. This is a good mindset in software development and very similar to the lean mindset.

First make something that works and only after that try to improve it.

## Improve problem-solving skills

Competitions create a lot of opportunities to exercise generating solutions, testing and validating them, identifying which ones do not fit, fixing them, and rethinking them. It helps a lot if you are not only able to quickly come up with solutions, but you are also able to quickly choose the one that/which best fits some defined criteria.

All contest problems are made so that they have an optimal solution and this solution can be discovered, implemented, and tested in the allotted time. Notice that solving a problem requires the following sequence of actions: read the problem, understand what it asks from you, mentally model what real-life situation the problem fits to, come up with solution ideas, quickly evaluate them and choose one, and make a plan of how this solution will look like in code, make a plan of what tests it must pass, write the code, fix the bugs, then test it some more. It could be of help if shortly before the contest you go through these steps by solving some random problem, similar to how athletes warm up shortly before a game.

## Playing with the programming language

Each contest is an opportunity to better understand your programming language, and to use it as much as possible, in a very pure and direct way. This is a good way for newcomers to learn about their programming language and for experienced developers to dust off some not so often used parts of it. In a contest, you are there facing a problem, having the language as your main tool, with little time to search for another solution or to learn a new API or a new module. You will need to make the most out of what you know and out of your previous experience with that programming language.

It is tempting to Google things about syntax or solutions to small intermediary problems. However, the time it takes to do so must be carefully considered, as in a contest you have little time to spare.

## Bugs and bug fixing

Testing and bug fixing is much harder in a contest compared to everyday work. You cannot afford to unit test and you really should not use a debugger unless you are sure you can afford the time. The bulk of your testing and bug fixing will be done most likely by simply looking at the code, reading it line by line, and thinking if it makes sense and what bugs could be hidden in that line of code. It is also a good example of trying to focus on the essential and improving your skills of quickly detecting a bug before execution.

A nice exercise to do when practicing for a contest is to use a site like [HackerRank](https://www.hackerrank.com/), [Codewars](https://www.codewars.com/), [TopCoder](https://www.topcoder.com/) and just look at various submitted solutions that pass most but not all test cases. Practice how to quickly spot bugs. This could be done even better if you have a training partner with whom you can review the code.

## Solving problems with your code

In the end, a contest is about focusing on the core, the essence of being a programmer: writing code that solves a given problem. No GUI, no API to integrate, not many docs to be read. It is only you and a problem, a keyboard, your favorite IDE, and your mind.

Coding contests are abundant, there probably is an online contest every single day. This should be a strong argument in favor of treating them like something fun, a training experience instead of a competition. In the end, the one who gains the most out of a given contest is the one who learned and improved the most.

## Meeting like-minded (or passionate) people

This is the right place to meet people who are passionate about coding, solving problems, optimizations, and about the same programming language as you are. It is the best group in which you can talk about IT, libraries, frameworks, about how to use a language quirk in a specific way.

---

Let’s end this (long) article with a bit of very practical advice for winning or at least increasing your chances of winning:

## Prepare a cheat sheet

It always helps to have some favorite algorithms and tricks printed out on paper. During the contest, you want to spend as little time as possible thinking about algorithms or researching. This list could be from very small, containing only some basic tricks, to extremely large, containing notes about almost all algorithms you ever encountered while training.

For example, here are three such possible notes (taken from [www.geeksforgeeks.org](http://www.geeksforgeeks.org/)):

[MergeSort](https://www.geeksforgeeks.org/merge-sort/)(arr[], l, r) — divides input array into two halves, calls itself for the two halves and then merges the two sorted halves.

```
If r > l
  1. Find the middle point to divide the array into two halves: 
middle m = (l+r)/2
  2. Call mergeSort for first half: Call mergeSort(arr, l, m)
  3. Call mergeSort for second half: Call mergeSort(arr, m+1, r)
  4. Merge the two halves sorted in step 2 and 3: Call merge(arr, l, m, r)
```

[Binary Search](https://www.geeksforgeeks.org/binary-search/) — searches a sorted array by repeatedly dividing the search array interval in half.

```
1. Compare x with the middle element.
2. If x matches with the middle element, we return the mid index.
3. Else If x is greater than the mid element, then x can only lie in the right half subarray after the mid element. So we recur for the right half.
4. Else (x is smaller) recur for the left half.
```

[Longest Increasing Subsequence](https://www.geeksforgeeks.org/longest-increasing-subsequence/) — find the length of the longest subsequence of a given sequence such that all elements of the subsequence are sorted in increasing order.


```
1. Let arr[0..n-1] be the input array and L(i) be the length of the LIS ending at index i such that arr[i] is the last element of the LIS.
2. Then, L(i) can be recursively written as:
   
 a. L(i) = 1 + max( L(j) ) where 0 < j < i and arr[j] < arr[i]; or
 b. L(i) = 1, if no such j exists.
3. To find the LIS for a given array, we need to return max(L(i)) where 0 < i < n.
```

---

This post was originally published on [METRO SYSTEMS Romania](https://medium.com/metrosystemsro) publication on Medium in January, 2018.

