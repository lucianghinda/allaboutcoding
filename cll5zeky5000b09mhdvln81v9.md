---
title: "Bugs, errors and causes from an 1975 paper"
seoTitle: "1975 Paper: Bugs, Errors, Causes Analysis"
seoDescription: "1975 paper: analyzing system program bugs, stressing problem definition, domain knowledge, diverse error categories for software quality"
datePublished: Fri Aug 11 2023 02:41:12 GMT+0000 (Coordinated Universal Time)
cuid: cll5zeky5000b09mhdvln81v9
slug: bugs-errors-and-causes-from-an-1975-paper
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1691721296253/2e2e77e5-39f4-42c0-98cc-56a3948d38ab.png
tags: bugs-and-errors, errors, programming-tips

---

Discover the timeless relevance of a 1975 paper that sheds light on the causes of bugs and errors in system programs.

In this article, we delve into Albert Endres' findings and explore how understanding the problem, effective communication, and domain knowledge play crucial roles in reducing errors.

These principles remain highly relevant in current software development practices, where collaboration, clear communication, and comprehensive knowledge of the project's domain could be crucial for the success of the product.

### About the paper

In 1975, [Albert Endres](https://ieeexplore.ieee.org/author/37087903653) published a paper called ["An Analysis of Errors and their Causes in System Programs"](https://ieeexplore.ieee.org/document/6312834).

In this paper, they analyze the errors detected in an operating system DOS/VS developed by IBM Laboratories in Germany.

The system was released in 1973. The paper analyses:

\- 500 modules, with an average of 360 lines per module and 480 lines of comments, resulting in 190K instructions and 60K comments.

\- A total of 740 problems were found, of which 432 were classified as program errors.

### Areas where errors where found

And here are some of the conclusions from this paper:

!["Almost half of all errors are found in the area of understanding the problem, problem communication and of the knowledge of possibilities and procedures for problem-solving"](https://cdn.hashnode.com/res/hashnode/image/upload/v1691661579216/060b5daa-657b-41c9-bbfe-a40cccd1ae2c.png align="center")

If we want to reduce the number of errors then we should focus not only on better programming techniques but also on the problem definition, and understanding domain knowledge:

!["This fact is alarming or encouraging, depending on the expectations we had for a hundred percent automation of software production.  More specifically, only half of the mistakes can be avoided with better programming techniques (better programming languages, more comprehensive test tools). The other half must be attacked with better methods of problem definition (specification languages), a better understanding of basic system concepts (training, education), and by making applicable algorithms available"](https://cdn.hashnode.com/res/hashnode/image/upload/v1691661619042/bed763a4-6280-4892-a686-2b8d96a2eb9a.png align="center")

And based on the paper here are the categories of causes for errors or bugs:

1\. Technological = *"definability of the problem, feasibility of solving it, available procedures and tools"*

2\. Organisational = *"division of work load, available in- formation, communication, resources"*

3\. Historic = *"history of the project, of the program, special situations, and external influences"*

4\. Group dynamic = *"willingness to cooperate, distribution of roles inside the project group"*

5\. Individual = *"experience, talent, and constitution of the individual programmer"*

6\. Other

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691661730037/ac6a15bd-2a23-4328-8c10-9b74359a0f7a.png align="center")

## Some conclusions:

1. When considering quality in a software product, we should consider all aspects, not only the tools and programming techniques we use.
    
2. When joining a product or team, it is important to understand the history of the product & team.
    
3. To become a better developer one has to learn more than coding/engineering techniques.
    
4. When we think about a possible future of using AI to augment programming, we should consider that if this study remains true it is not enough to reduce the number of errors. As half of them are not caused by code.