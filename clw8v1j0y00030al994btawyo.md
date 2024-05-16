---
title: "Add code snippet in RubyMine and Visual Studio Code to binding.break"
seoTitle: "Add Binding.break code snippet in RubyMine & VS Code"
seoDescription: "Learn how to add a code snippet for adding binding.break in RubyMine and Visual Studio Code"
datePublished: Thu May 16 2024 06:19:09 GMT+0000 (Coordinated Universal Time)
cuid: clw8v1j0y00030al994btawyo
slug: add-code-snippet-in-rubymine-and-visual-studio-code-to-bindingbreak
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1715840276608/529af8a1-0cf5-4a83-8bfa-1c3548f8ea2e.png
tags: ruby, ruby-on-rails, vscode, rubymine, codesnippet

---

Here is a quick way to add a manual breakpoint in the code using code snippets.

## The code

What I usually want to see directly when I debug Ruby code and I cannot (or don't have time to set up the RDBG) is the variables and methods of the current object and the values of all instance variables and local variables at the breakpoint.

For this, I moved to use the [`debug`](https://github.com/ruby/debug) gem, and I wrote:

`binding.break(pre: 'ls ;; i')`

Where ([source](https://github.com/ruby/debug?tab=readme-ov-file#information)):

* `ls` - *"Show you available methods, constants, local variables, and instance variables in the current scope"*
    
* `i` - *"Show information about current frame (local/instance variables and defined constants)"*
    

## RubyMine

In RubyMine, code snippets are called *Live Templates*; you can find them in Settings.

![How to add a Live Template in RubyMine](https://cdn.hashnode.com/res/hashnode/image/upload/v1715830465857/9fc49b08-db65-46e3-a25b-24c00e047442.png align="center")

The steps to add a code snippet are:

1. Search for `Live Templates`
    
2. Open Live Templates and click on the `+` sign to add a new Live Template
    
3. Fill in the `Abbreviation` with `debug` (or if you prefer a shorter char choose something else)
    
4. Fill in the `Template Text` with `binding.break(pre: 'ls ;; i')`
    
5. Choose the expand action - I use `Tab` for it
    
6. Write a meaningful description for you. I wrote here: `Debug with binding.break`
    
7. Click `Apply`
    

Here is what it looks like when run:

%[https://youtu.be/KKJ4MC3gzc4] 

## Visual Studio Code

To add the same snippet to Visual Studio Code you have to do the following steps:

1. Go to Settings -&gt; Configure User Snippets
    
2. Choose "New Global Snippets File" and give it a name like `Ruby Snippets`
    
3. And then add the following inside that file:
    

```json
{
  "Set breakpoint with ruby debug gem": {
    "prefix": "debug",
    "body": [
        "binding.break(pre: 'ls ;; i')"
    ],
    "description": "Insert binding.break with outline and info about current frame"
  }
}
```

Save it and it will just work.

Here is how it looks like when you are using it:

%[https://youtu.be/yKbxIMGX7fg] 

And that's it.

## A better alternative

A better alternative for debugging is to use in **RubyMine** the [integrated debugger](https://www.jetbrains.com/help/ruby/debugging-code.html) or to use the [**VSCode rdbg Ruby Debugger**](https://marketplace.visualstudio.com/items?itemName=KoichiSasada.vscode-rdbg) in Visual Studio Code.

I am using the code snippet when I want a quick jump in to execute some custom code and the project is not configured to work with any of those debuggers. I sometimes take a look at open source Ruby on Rails projects and I want the fastest way to look at what's happening in some specific points in code.