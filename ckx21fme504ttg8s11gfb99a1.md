## Minimize dependencies when possible

This is more like general advice, but when possible minimize dependencies. 

I will write the following examples with Ruby gems in mind but could be used in any other programming environment that supports packages. 

## Why minimize external dependencies 

Of course, there are pros and cons to using external packages. 

What are some advantages of using external dependencies: 
1. Delegating to library maintainers/creators to make the best decisions and assume they are good in the library domain. 
2. Delegate maintenance to maintainers
3. Delegate review and bug fixing to people using the library thus in general a library could be better covered with tests or production usage than your own code. 

In general, I think you are using a library you should consider contributing to it in any way you can. 

But let's talk about disadvantages:
1. Security: you add a new vector of attack
2. Possible upgrade issues: if you decide to upgrade your OS or language then you need to make sure that the library is supported. You can and should at any time you upgrade and find a library not working contribute to it and propose a patch. But sometimes this is not easy to do. 


## Decision making: when to use a library and when not

I am using a simple heuristic to decide when to use a library and when to implement your own code.


### When to write your own code


As a general rule: 

> Write your own code if the code that you are about to write is simple.

Also, write your code: 

1. When the solution would **fit in a single method/function**
2. When **the solution is simple** and it seems like something that the standard library should support
3. When **you know a lot about the domain** of the solution and you can implement a simple solution
4. When the solution is simple and **importing a library will include a lot of other dependencies**
5. When **you will need to change, adapt, monkey patch an existing library to fit your case**. 


### When not to write your own code

As a general rule: 

> Do not write your own code if the code you are about to write could be something complex or is in a domain that could have multiple unknowns and there is already a library providing a solution

Do not write your own code:

1. If the solution will be something complex that will require building a small library => in this case better to find an existing library that mostly matches what you need and contribute to it or fork it. For example: do not implement your own email validator unless you are willing to read domain and email RFCs. 
2. If the solution is something that might have a lot of edge cases => the chances are if you are not an expert in the domain that you will miss some cases to it is better to delegate. For example: do not implement a library that would handle image/video processing as this is a domain that has a lot of edge cases. 
3. If the code that you are about to implement will require you to understand a new domain => the chances are that you might not know what you don't know so better to delegate. For example: do not implement a library to handle international taxes if you are not willing to go deep into this and have time to understand various rules countries have in place. 
4. If your solution has any security implications and you are not an expert in security => here I think it is better to delegate to well-known, maintained, and uses libraries. Example: do not roll your own authentication system if you don't fully understand what this means and what security best practices are required in this case








