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

As I was working on another email part of my [Modern Ruby Email Course](https://learn.shortruby.com/courses/modern-ruby) I wanted to make some micro benchmarks on `Data.define` vs `Struct` vs `OpenStruct`

They are not a production level benchmark so take it with a grain of salt.

I added all code and results in a repo at [https://github.com/lucianghinda/value-object-in-ruby-benchmarks](https://github.com/lucianghinda/value-object-in-ruby-benchmarks)

## Creating new objects

When creating a new object `Struct.new` is the fastest one looking at all 3 tests and `OpenStruct` is the slowest one.

Here is a `` `bmbm` `` benchmark result:

```bash
Creating a new object - Benchmark with bmbm
Rehearsal --------------------------------------------------
Struct.new       0.000005   0.000000   0.000005 (  0.000004)
Data.define      0.000042   0.000001   0.000043 (  0.000041)
OpenStruct.new   0.001642   0.000075   0.001717 (  0.001718)
----------------------------------------- total: 0.001765sec

                     user     system      total        real
Struct.new       0.000004   0.000001   0.000005 (  0.000004)
Data.define      0.000044   0.000000   0.000044 (  0.000044)
OpenStruct.new   0.001061   0.000015   0.001076 (  0.001079)
```

Here is the `ibs` benchmark result:

```bash
Creating a new object - Benchmark with ips
ruby 3.3.0 (2023-12-25 revision 5124f9ac75) [arm64-darwin23]
Warming up --------------------------------------
          Struct.new    36.826k i/100ms
         Data.define     2.613k i/100ms
      OpenStruct.new    57.000 i/100ms
Calculating -------------------------------------
          Struct.new    376.794k (±10.1%) i/s -      1.878M in   5.039312s
         Data.define     25.731k (± 1.7%) i/s -    130.650k in   5.078945s
      OpenStruct.new    584.608 (± 1.2%) i/s -      2.964k in   5.070709s

Comparison:
          Struct.new:   376794.1 i/s
         Data.define:    25731.1 i/s - 14.64x  slower
      OpenStruct.new:      584.6 i/s - 644.52x  slower
```

Here is the `memory` benchmark result:

```bash
Creating a new object - Benchmark with ips
Calculating -------------------------------------
          Struct.new     8.040k memsize (     0.000  retained)
                         1.000  objects (     0.000  retained)
                         0.000  strings (     0.000  retained)
         Data.define    36.792k memsize (     0.000  retained)
                         2.000  objects (     0.000  retained)
                         0.000  strings (     0.000  retained)
      OpenStruct.new   791.224k memsize (     0.000  retained)
                         8.003k objects (     0.000  retained)
                        50.000  strings (     0.000  retained)

Comparison:
          Struct.new:       8040 allocated
         Data.define:      36792 allocated - 4.58x more
      OpenStruct.new:     791224 allocated - 98.41x more
```

## Accessing attributes

This time `Data.define` is maybe the fastest one but there is not a signifiant difference to `Struct`. On the other side `OpenStruct` is almost twice as slow.

Here is the `bmbm` benchmark result:

```bash
Accessing attributes - bmbm test
Rehearsal -----------------------------------------------
Struct        0.000070   0.000002   0.000072 (  0.000070)
Data.define   0.000064   0.000002   0.000066 (  0.000065)
OpenStruct    0.000108   0.000005   0.000113 (  0.000113)
-------------------------------------- total: 0.000251sec

                  user     system      total        real
Struct        0.000044   0.000001   0.000045 (  0.000044)
Data.define   0.000042   0.000000   0.000042 (  0.000042)
OpenStruct    0.000090   0.000001   0.000091 (  0.000089)
```

Here is the `ibs` benchmark result:

```bash
Accessing attributes - ips test
ruby 3.3.0 (2023-12-25 revision 5124f9ac75) [arm64-darwin23]
Warming up --------------------------------------
              Struct     2.741k i/100ms
         Data.define     2.799k i/100ms
          OpenStruct     1.370k i/100ms
Calculating -------------------------------------
              Struct     27.832k (± 0.2%) i/s -    139.791k in   5.022750s
         Data.define     27.989k (± 0.3%) i/s -    139.950k in   5.000245s
          OpenStruct     13.434k (± 0.5%) i/s -     68.500k in   5.099337s

Comparison:
         Data.define:    27988.9 i/s
              Struct:    27831.7 i/s - 1.01x  slower
          OpenStruct:    13433.5 i/s - 2.08x  slower
```

## Curious to learn more about this?

If you want to learn more about `Data.define`, its gotchas, when and how to use it, and real code examples, I am writing a course called [“Modern Ruby Syntax”](https://learn.shortruby.com/courses/modern-ruby), where I explore the `Data.define` along with other new Ruby features.

---

**Enjoyed this article?**

👐 Subscribe to my Ruby and Ruby on rails courses over email at [**learn.shortruby.com**](http://learn.shortruby.com/)**\- effortless learning about Ruby anytime, anywhere**

👉 Join my [**Short Ruby News**](https://shortruby.com/)**newsletter** for weekly Ruby updates from the community and visit [**rubyandrails.info**](http://rubyandrails.info/)**, a directory with learning content about Ruby.**

🤝 Let's connect on [**Ruby.social**](http://ruby.social/) or [**Linkedin**](https://linkedin.com/in/lucianghinda) or [**Twitter**](https://x.com/lucianghinda) where I post mainly about Ruby and Rails.

🎥 Follow me on [**my YouTube channel**](https://www.youtube.com/@shortruby) for short videos about Ruby