---
title: "Ruby Gems Download Trends: An Analysis from 2013 to 2023"
seoTitle: "Ruby Gems Trends: 2013-2023 Analysis"
seoDescription: "Analyze Ruby Gems download trends (2013-2023), showing rapid growth from 2019, likely linked to CI/CD adoption and Ruby's rising popularity"
datePublished: Wed Nov 08 2023 03:16:22 GMT+0000 (Coordinated Universal Time)
cuid: clop6umau000708l0gbrw5r3y
slug: ruby-gems-download-trends-an-analysis-from-2013-to-2023
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1699270501957/e58a4923-cffa-41b1-915d-a697e4460d36.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1699413334318/4bb7097c-0ff5-4562-9b84-99f0f857e299.png
tags: programming-blogs, ruby, ruby-on-rails, coding, programming-languages

---

While creating the Short Ruby Newsletter I discovered [bestgems.org](https://bestgems.org) and I noticed there was an interesting graph and would like to discuss it a bit.

# Downloads trends of all gems

![A graph showing downloads trends of all gems](https://cdn.hashnode.com/res/hashnode/image/upload/v1699187477990/f3d7a87e-6c9f-4aee-814e-6f49e94461e9.png align="center")

[source](https://bestgems.org/stat/downloads)

## Data points

Here is how I read this, first there are two data sets: the total downloads and the daily downloads

**Total downloads**

It is a sum of downloads of all gems -&gt; so of course this will always be increasing.

But what I think is interesting is how fast the total downloads are increasing. I think it can be seen with the naked eye that starting with say 2019/2020 the total downloads are increasing:

* **1.47B** downloads in 2013-07-1 and **3.71B** downloads in 2019-07-01 = **a difference of ~2B downloads increased in 5 years**
    
* and then we have **137.53B** downloads in 2023-07-01 = **a difference since 2019 of ~135B downloads in 4 years**
    

*Conclusion:* There are two orders of magnitude increase in the total downloads in the last 4 years versus the previous 5 years.

**Daily downloads**

If the total downloads are increased of course also daily downloads are increased.

In 2013 the graph starts with **5.75 M** downloads per day (in 2013-07-03). Then we have **50M** downloads per day in 2019-07-03. And then **127M** downloads per day on 2023-07-11.

Conclusion: the number of downloads per day is more than double, with some periods from 2019 until 2023 with a higher number of downloads per day (like 160M or 154M).

## What does this mean?

**For sure it means the number of downloaded gems accelerated since 2019.**

**One explanation** could be running tests using CI pipelines and containers so that for every run there is a download of gems.

Some data points:

* GitHub launched its Github Actions in 2018 ([source](https://resources.github.com/devops/tools/automation/actions/#))
    
* Gitlab launched its first version of CI in 2012 ([source](https://handbook.gitlab.com/handbook/company/history/#2012-gitlabcom))
    
* Jenkins launched in 2011 ([source](https://www.cloudbees.com/jenkins/what-is-jenkins))
    
* Circle CI launched in 2011 ([source](https://circleci.com/careers/))
    
* Travis CI launched in 2011 ([source](https://www.travis-ci.com/about-us/))
    
* Almost 30% of businesses were already using CI/CD in 2013-2014 ([source](https://www.apexon.com/blog/the-road-to-cicd-a-short-history-of-agile-development/))
    

I will assume these features took off the next year so it would be 2019 for Github and 2013 for Gitlab and the other major CIs 2012.

It is hard to know if this explanation is the only one. I don't feel it is accounting for all the fast growth since 2019. The only data point that matches that is Github launching their Actions in 2019 but that alone cannot explain the increase of two orders of magnitude since then.

**Another explanation** could be that Ruby is growing and the pace is increasing. There could be two root causes for this:

* More often version updates for gems result in multiple organisations' teams updating
    
* The pace of new projects started is increasing
    

Again it is hard to know exactly which one is the reality. But I add this explanation that Ruby is growing because I see signs of it also in other places (more conferences, more talk on social media, an increased number of articles, more releases, more jobs, more contacts on Linkedin about jobs ...)

**There can be other causes for the increase** we have seen since 2019 - maybe the way data is gathered changed, it is more granular or better captured.

I think it is a combination of both explanations - CI being adopted on a large scale while also the Ruby number of projects growing more rapidly in the last 3-4 years.

## Data validation

[**bestgems.org**](https://bestgems.org) is not [rubygems.org](https://rubygems.org) and I could not yet find how they are taking their data.

Still, when looking at the total number of downloads (see the comparison image below), it seems to have the same data so I wrote this article starting with the assumption that bestgems.org data is valid.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699273394712/48c741aa-ebdb-467e-9106-4433399e1160.png align="center")

(left) [https://rubygems.org/stats](https://rubygems.org/stats) vs (right) [https://bestgems.org/stat/downloads](https://bestgems.org/stat/downloads)

## Maybe a conclusion

The rapid increase in Ruby gem downloads since 2019 can be attributed to the widespread adoption of CI/CD pipelines (with a small note that a lot of CI/CD tools were launched years before) and also to the growing popularity of Ruby in new projects.

This trend reflects the growing importance of Ruby in the software development ecosystem.