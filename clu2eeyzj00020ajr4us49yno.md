---
title: "Micro benchmarking value objects in Ruby: Data.define vs Struct vs OpenStruct"
seoTitle: "Ruby Benchmark: Data.define, Struct, OpenStruct"
seoDescription: "Compare `Data.define`, `Struct`, `OpenStruct` in Ruby: object creation and accessing attributes"
datePublished: Fri Mar 22 2024 08:27:41 GMT+0000 (Coordinated Universal Time)
cuid: clu2eeyzj00020ajr4us49yno
slug: micro-benchmarking-value-objects-in-ruby-datadefine-vs-struct-vs-openstruct
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1711095989974/54d8b470-89df-4ead-9f6c-d26d5c75680d.png
tags: programming-blogs, ruby, performance, ruby-on-rails

---

As I was working on another email part of my [Modern Ruby course via email](https://learn.shortruby.com/courses/modern-ruby) I wanted to make some micro benchmarks on `Data.define` vs `Struct` vs `OpenStruct`

They are not a production-level benchmark, so take them with a grain of salt.

I added all code and results in a repo at [https://github.com/lucianghinda/value-object-in-ruby-benchmarks](https://github.com/lucianghinda/value-object-in-ruby-benchmarks)

## Creating new objects

When creating a new object, Struct (with keyword\_init)and Data.define behave almost the same (the differences are with error margin or so small that they are probably due to my setup), while `OpenStruct` seems to be the slowest one.

Here is a `` `bmbm` `` benchmark result:

```bash
Creating a new object - Benchmark with bmbm
Rehearsal --------------------------------------------------
Struct.new       0.000023   0.000003   0.000026 (  0.000024)
Data.define      0.000020   0.000001   0.000021 (  0.000022)
OpenStruct.new   0.001705   0.000075   0.001780 (  0.001780)
----------------------------------------- total: 0.001827sec

                     user     system      total        real
Struct.new       0.000020   0.000000   0.000020 (  0.000020)
Data.define      0.000022   0.000000   0.000022 (  0.000022)
OpenStruct.new   0.001069   0.000044   0.001113 (  0.001132)
```

Here is the `ibs` benchmark result:

```bash
Creating a new object - Benchmark with ips
ruby 3.3.0 (2023-12-25 revision 5124f9ac75) [arm64-darwin23]
Warming up --------------------------------------
          Struct.new     5.169k i/100ms
         Data.define     5.361k i/100ms
      OpenStruct.new    62.000 i/100ms
Calculating -------------------------------------
          Struct.new     50.086k (± 1.7%) i/s -    253.281k in   5.058450s
         Data.define     51.646k (± 1.1%) i/s -    262.689k in   5.086990s
      OpenStruct.new    607.447 (± 0.8%) i/s -      3.038k in   5.001584s

Comparison:
         Data.define:    51646.3 i/s
          Struct.new:    50085.7 i/s - 1.03x  slower
      OpenStruct.new:      607.4 i/s - 85.02x  slower
```

Here is the `memory` benchmark result.

```bash
Creating a new object - Benchmark with ips
Calculating -------------------------------------
          Struct.new    36.792k memsize (     0.000  retained)
                         2.000  objects (     0.000  retained)
                         0.000  strings (     0.000  retained)
         Data.define    36.792k memsize (     0.000  retained)
                         2.000  objects (     0.000  retained)
                         0.000  strings (     0.000  retained)
      OpenStruct.new   848.728k memsize (     0.000  retained)
                         8.005k objects (     0.000  retained)
                        50.000  strings (     0.000  retained)

Comparison:
          Struct.new:      36792 allocated
         Data.define:      36792 allocated - same
      OpenStruct.new:     848728 allocated - 23.07x more
```

## Accessing attributes

Again `Data.define` and `Struct` with keyword arguments are the same. On the other side `OpenStruct` is almost twice as slow.

Here is the `bmbm` benchmark result:

```bash
Accessing attributes - bmbm test
Rehearsal -----------------------------------------------
Struct        0.000069   0.000002   0.000071 (  0.000071)
Data.define   0.000069   0.000003   0.000072 (  0.000071)
OpenStruct    0.000110   0.000003   0.000113 (  0.000116)
-------------------------------------- total: 0.000256sec

                  user     system      total        real
Struct        0.000049   0.000001   0.000050 (  0.000046)
Data.define   0.000046   0.000001   0.000047 (  0.000046)
OpenStruct    0.000091   0.000001   0.000092 (  0.000094)
```

Here is the `ibs` benchmark result:

```bash
Accessing attributes - ips test
ruby 3.3.0 (2023-12-25 revision 5124f9ac75) [arm64-darwin23]
Warming up --------------------------------------
              Struct     2.857k i/100ms
         Data.define     2.828k i/100ms
          OpenStruct     1.384k i/100ms
Calculating -------------------------------------
              Struct     28.420k (± 0.9%) i/s -    142.850k in   5.026906s
         Data.define     28.691k (± 0.5%) i/s -    144.228k in   5.027131s
          OpenStruct     13.475k (± 0.9%) i/s -     67.816k in   5.033315s

Comparison:
         Data.define:    28690.8 i/s
              Struct:    28419.6 i/s - same-ish: difference falls within error
          OpenStruct:    13474.6 i/s - 2.13x  slower
```

## Context for understanding why Data.define and Struct are similar

[Ufuk Kayserilioglu](https://ruby.social/@ufuk) [explains](https://ruby.social/@ufuk/112141972493634321) why `Data.define` and `Struct` with keyword arguments have the same behavior:

![Ufuk explaining: "Data is (basically) just Struct with no writer methods defined (and a freeze, I believe). The CRuby codepaths are exactly the same for both, except zverok decided that Data#initialize should always accept kw arguments, so Data.new has to convert positional args to kw args before passing them to initialize."](https://cdn.hashnode.com/res/hashnode/image/upload/v1711170357937/75fad78f-b200-436d-93c3-4b59a2e96318.png align="center")

## A note about `OpenStruct`

[Jean Boussier](https://github.com/byroot) [answered](https://twitter.com/_byroot/status/1771221003051422101) a question about why OpenStruct is so slow:

![Jean Boussier explaining "It's not because it's written in Ruby but because of it's semantic.  For every single instance it has to create a metaclass and define methods on it. It's terribly wasteful and doing the same in C wouldn't be much faster.  OpenStruct should be considered deprecated really."](https://cdn.hashnode.com/res/hashnode/image/upload/v1711170068681/4d2394ac-51f8-4e8e-b7a2-7e84d69235b1.png align="center")

## Updates

This article was updated on 2024-03-23 after [**Ufuk Kayserilioglu**](https://twitter.com/paracycle) added a PR to the repo after he [noticed](https://twitter.com/paracycle/status/1771178501015580769) an error in my benchmarking where I was comparing Data.define with Struct with positional arguments. And I also added explanation shared by Jean Boussier about the slowness of OpenStruct and explanation by Ufuk about the similar behaviour of Data.define and Struct.

---

**Enjoyed this article?**

👐 Subscribe to my Ruby and Ruby on Rails courses over email at [**learn.shortruby.com**](http://learn.shortruby.com/)**\- effortless learning about Ruby anytime, anywhere**

👉 Join my [**Short Ruby News**](https://shortruby.com/)**newsletter** for weekly Ruby updates from the community and visit [**rubyandrails.info**](http://rubyandrails.info/)**, a directory with learning content about Ruby.**

🤝 Let's connect on [**Ruby.social**](http://ruby.social/) or [**Linkedin**](https://linkedin.com/in/lucianghinda) or [**Twitter**](https://x.com/lucianghinda) where I post mainly about Ruby and Rails.

🎥 Follow me on [**my YouTube channel**](https://www.youtube.com/@shortruby) for short videos about Ruby