# In defense of Ruby's short-hand syntax

*This is primarily a response* [*to a Reddit comment*](https://www.reddit.com/r/rails/comments/10f09zp/comment/j4uev23/?context=3) *but I have encountered this line of pushback against short-hand syntax multiple times. Thus I think it is good to write it down and explain my support for short-hand syntax or, in general, for new language features*

First: I disagree that shorthand syntax is code golfing. It makes the code shorter, but in my mind, it does not take away readability.

Second, in support of the short-hand syntax in my argument, my main argument is that people don't like it because they are not familiar with it. It is new thus not used that much in codebases.

## A methaphor

Using shorthand syntax is very similar to, for example, using `attr_accessor: var` - it means understanding something abstract because we rely on a-priory information that we take for granted in the programming language.

Taking it for granted means we assume it is readable because we learned the concept, and we can recognize it even when it is described succinctly, and moreover this kind of code is present in many places thus, it is very familiar.

Here are other examples: `flat_map` or even `map` are features that were once new, but we learned how they work, we see them constantly and we now accept them as being readable because we are familiar with them.

Thus this shorthand syntax hides complexity into a bigger abstraction. The same way `attr_accessor` is doing.

I can easily imagine 20 years ago someone saying:

> I don't want to use `attr_accessor` because it is not clear that the object will have `def var=` and `def var` methods defined.

But as we become more and more comfortable with the new information (i.e. knowing that when we write`attr_accessor` we are creating the getters and setters), we accept it as being readable.

## About short-hand syntax

I think the same goes for this line:

```ruby
render locals: { user: }
```

It is a shorthand of:

```ruby
render locals: { user: user }
```

And the moment we upload/know/recall in our mind that `{ user: }` means that there is a variable or method named `user` defined - it is just another way to hide complexity and provide brevity.

## In defense of breavity

Regarding the idea of readability + Ruby + better code, here the opinions can be subjective.

Thus I will rely on Matz words from "Treating Code As an Essay":

First, to lay out a bit the foundation that Matz is talking about readable code as being beautiful code:

> “Computers can, of course, deal with complexity without complaint, but this is not the case for human beings. Unreadable code will reduce most people's productivity significantly. On the other hand, easily understandable code will increase it. And we see beauty in such code.”
> 
> \[...\]
> 
> “beautiful code is really meant to help the programmer be happy and productive. This is the metric I use to evaluate the beauty of a program.”

and then here is what he says about what makes code beautiful:

> “Brevity is one element that helps make code beautiful. As Paul Graham says, "Succinctness is power." In the vocabulary of programming, brevity is a virtue. Because there is a definite cost involved in scanning code with the human eye, programs should ideally contain no unnecessary information.”
> 
> \[...\]
> 
> “**Brevity can also mean the elimination of redundancy**. Redundancy is defined as the duplication of information. When information is duplicated, the cost of maintaining consistency can be quite high. And because a considerable amount of time can be spent maintaining consistency, redundancy will lower programming productivity.”

## Short-hand syntax bring breavity

Thus for me, the short-hand syntax is a beautiful example of the elimination of redundancy:

**We don't need to repeat the same name**

```ruby
render locals: { user: user, records: records, page: page }
```

and

**We don't need to rely on naming tricks when we don't want to repeat the name like prefixes** `get_` `fetch_` ...

```ruby
render locals: { user: fetch_user, records: fetch_records }
```