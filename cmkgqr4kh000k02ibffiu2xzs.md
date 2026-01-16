---
title: "Improving Git Diffs with Delta"
seoTitle: "Improve Git Diffs Using Delta"
seoDescription: "Use Delta to improve Git diffs with syntax highlighting for better readability in your terminal. Install and configure for better code reviews"
datePublished: Fri Jan 16 2026 10:34:11 GMT+0000 (Coordinated Universal Time)
cuid: cmkgqr4kh000k02ibffiu2xzs
slug: improving-git-diffs-with-delta
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1768559571890/5d2ae8bc-6429-4087-afd6-1bba3945574e.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1768559629898/12a454c6-571c-48b9-ae79-06e43a299326.png
tags: ai, programming-blogs, git, llm

---

Working with LLMs means reviewing way more diffs than before. I discovered delta a while back and this was such a huge improvement to working in a terminal and having amazing syntax highlighting for diffs.

If you spend significant time reviewing code changes in the terminal, delta can transform your workflow from reading simple diff output to working with syntax-highlighted, readable diffs.

## Installing and Configuring Delta

First, install delta from [https://github.com/dandavison/delta](https://github.com/dandavison/delta)

Then configure your `.gitconfig`:

```yaml
[core]
    pager = delta

[interactive]
    diffFilter = delta --color-only

[delta]
    navigate = true
    dark = true

[merge]
    conflictstyle = zdiff3
```

## Visual Comparison

The difference is immediately visible when you run `git diff`. Without delta, you get standard monochrome output that can be hard to parse, especially for longer diffs. With delta, you get:

* Syntax highlighting based on file type
    
* Side-by-side or unified view options
    
* Line numbers
    
* Better visual separation between changes
    

![Side-by-side comparison of two git diffs: plain monochrome diff labeled "without delta" and colorful highlighted diff labeled "with delta," showing test file changes.](https://cdn.hashnode.com/res/hashnode/image/upload/v1768538913060/100bf638-65da-479d-9689-2251303f86a7.png align="center")

## Configuring Lazygit with Delta

I use [lazygit](https://github.com/jesseduffield/lazygit) for managing my git workflow. Configuring it to work with delta is simple, just add this line to your lazygit configuration:

```yaml
git:
  paging:
    pager: delta --dark --paging=never --line-numbers
```

## The Result

Without delta, lazygit displays diffs in plain diff format with just a bit of highlighting. With delta configured, you get the same rich syntax highlighting and visual improvements directly in your lazygit interface.

Here is how lazygit looked without delta configured:

![Dark-themed terminal split into git status and diff panes showing file tree, branches, commits, and Ruby test code, titled "lazygit without delta."](https://cdn.hashnode.com/res/hashnode/image/upload/v1768538951035/bef75d4d-267c-4d78-9876-abb77a0f681c.png align="center")

And here is how it looks like with delta configured:

![Terminal-style git UI showing lazygit with delta-style colored diff panels: left panes for status, branches and commits; right pane with code lines and inline red/green change highlights.](https://cdn.hashnode.com/res/hashnode/image/upload/v1768538993568/895f7493-1483-485b-a871-bcc6aa621d2b.png align="center")

This makes reviewing changes faster and reduces eye strain when working through multiple diffs in a session. When you are working with LLMs and constantly reviewing generated code or refactorings, these small improvements in readability add up quickly.

---

👉 If you like this article and want it in your inbox each week, [**subscribe to my newsletter**](https://newsletter.lucianghinda.com/). You’ll find **ideas on Ruby, software development, software testing, building products and workshops**, plus notes on creativity, tech trends, and whatever else sparks my curiosity.

👐 Want to improve your **developer testing skills**? Visit [**goodenoughtesting.com/articles**](https://goodenoughtesting.com/articles) to discover resources on testing for developers.

👉 [**Join my Short Ruby Newsletter**](https://newsletter.shortruby.com/) for weekly Ruby updates and visit rubyandrails.info, a directory of Ruby learning content.

🤝 Connect with me on [**Linkedin**](https://linkedin.com/in/lucianghinda), [**Bluesky**](https://bsky.app/profile/lucianghinda.com), [**Ruby.social**](https://ruby.social/@lucian), , and [**Twitter**](https://x.com/lucianghinda), where I mostly post about Ruby and Ruby on Rails.

🎥 Follow [**my YouTube channel**](https://www.youtube.com/@shortruby) for short videos about Ruby and Rails.