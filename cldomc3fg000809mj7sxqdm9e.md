# We should adopt and use new Ruby features

### A bit of (personal) history

I learned Ruby, I think, around 2007, and back then, it was a language pushing boundaries for me. Before encountering Ruby, I worked with Java, C/C++, PHP, and Python, and even dig into the strange world of JS (back then when [Scriptaculous](https://madrobby.github.io/scriptaculous/) and [Prototype.js](https://en.wikipedia.org/wiki/Prototype_JavaScript_Framework) were the hot JS libraries and jQuery was appearing).

In the beginning, *Ruby looked strange, made people uncomfortable when reading the syntax, and created emotion*.

I remember when I read the first time code written in Ruby (1.8.6) was very strange for me: no `;`, optional parenthesis, using `unless`, being able to write `if` at the end of a statement, the strange lack of `for`, `while` , `repeat` and the continued usage of `each` this was just the tip of the iceberg along with Rails 1.2 and later on Rails 2.

Ruby looked very alien compared with other typical languages that I was using. For me, it was a language that created its own category, very close to the English language and with pseudocode.

### **Where are we now?**

In my opinion, we are a bit too conservative with adopting new features. This attitude is good for projects built with Ruby; it comes from experience and wisdom. We tried many things over all these years, and we settled on what works and does not.

We say Ruby is boring or Rails is boring, and that's good for organizations and their projects.

At the same time, I feel that we slowed the pace of innovation for code design in the projects built with, Ruby. We are still experimenting, but as far as I see (limited experience of N=1), the adoption rate for new features is slow. There are still features released three years ago (e.g Ruby 2.7 in 2019) for which we still have rules/guidelines against them or limiting usage. See as example numbered params.

Projects built with Ruby were walking in new territories, playing with how code looks, pushing the language boundaries, and bringing art and beauty into how code looks.

But then we settled. We wanted to protect the fantastic thing we discovered.

By protecting it, we are also taking away its light.

I want Ruby to experiment with more features. And I trust Matz and the other core committers that they still have the same great taste for a great language.

### Why use new features and language constructs

Learning a new language feature is very similar to learning new words, cultural constructions, or expressions in a speaking language.

Thus my argument for learning and using new features/new language concepts is based on the theory of linguistic relativity and the hypothesis of linguistic determinism.

*In the same way, words or the language (we speak) shape our thinking and our reality, programming language constructs or features shape how we define a problem and the solution that we code for that problem.*

Fundamentally, when we write code, we model reality into another universe that can be described with a limited/fixed set of words or constructs.

This modeling is very similar to speaking another language. The more you know from the secondary language (dictionary, grammar, rules, cultural compositions), the better you can express your thoughts. Of course, this goes the same with your maternal language.

The same goes for knowing a programming language. The more you know (and use) the entire set of language constructs/features, the better you can express your solution.

Here is an example:

* In Ruby - we have `&&` and `||` but we also have `and` and `or` They can be used in various contexts to express different intentions and to control the flow.
    
* If you decide only to use `&&` and `||` then the way you will think about the solution (the algorithm) will then be shaped only by how `&&` and `||` behave.
    
* Thus even if the reality that you want to moderate might be better expressed in some cases with `and` and `or` you will force it to fit into `&&` and `||`.
    
Talking specifically about Ruby, there are more examples of new features or changes from Ruby 2.7+ that, for me, are changing the way I write code or at least it changes (sometimes) the way I think about code design:
- [Pattern Matching]([](https://docs.ruby-lang.org/en/3.0/syntax/pattern_matching_rdoc.html))
- [Numbered block params](https://rubyreferences.github.io/rubychanges/2.7.html#numbered-block-parameters)
- [Beginless range](https://rubyreferences.github.io/rubychanges/2.7.html#beginless-range)
- [Endless methods](https://rubyreferences.github.io/rubychanges/3.0.html#endless-method-definition)
- [Arguments forwarding](https://rubyreferences.github.io/rubychanges/3.0.html#arguments-forwarding--supports-leading-arguments)
- [private, public, protected returning their arguments](https://rubyreferences.github.io/rubychanges/3.1.html#moduleprivate-public-protected-and-module_function-return-their-arguments)
- [The new Data class](https://zverok.space/blog/2023-01-03-data-initialize.html)
- [Kernel.load adding accepting new param to evaluate code in context of a module](https://rubyreferences.github.io/rubychanges/3.1.html#kernelload-module-as-a-second-argument)
- [Hash literal value omission](https://rubyreferences.github.io/rubychanges/3.1.html#values-in-hash-literals-and-keyword-arguments-can-be-omitted)
- [Enumerator.product]([](https://bugs.ruby-lang.org/issues/18685))

And this is not a comprehensive list of all the exciting things that are happening Ruby in the last years. 

---

In the end, here are some starting points about linguistic relativity and linguistic determinism:

* Lera Boroditsky, [How Language Shapes Thought](https://www.scientificamerican.com/author/lera-boroditsky/), Scientific American, 2011:
    

> "These are just some of the many fascinating findings of cross-linguistic differences in cognition. But how do we know whether differences in language create differences in thought, or the other way around? The answer, it turns out, is both—the way we think influences the way we speak, but the influence also goes the other way. The past decade has seen a host of ingenious demonstrations establishing that language indeed plays a causal role in shaping cognition. Studies have shown that changing how people talk changes how they think. Teaching people new color words, for instance, changes their ability to discriminate colors. And teaching people a new way of talking about time gives them a new way of thinking about it"

* Harriet Joseph Ottenheimer and Judith M.S. Pine, ***The Anthropology of Language: An Introduction to Linguistic Anthropology,* Fourth Edition,** 2018
    

> "What is important to recognize, even more than the idea that you might think or perceive the world differently depending on the language you speak, is that before you can really use a new language comfortably, without thinking about what you are saying, you need to wrap your mind around the new concepts that the new language is presenting to you"

> "According to cognitive linguists, the words we use create—and are used within—frames. The idea of frames is similar to the idea of worldview. We view the world through frames. Frames often invoke cultural metaphors, grouping ideas into commonly used phrases. As such, they often invoke an ideology, or a set of ideas we have about the way things should be.
> 
> \[...\]
> 
> It is more difficult to talk about, and perhaps even to think about, something that you have no frame for in your language. Cognitive scientists call this hypocognition, or lack of the ideas that you need for talking or thinking about something"

---

If you like this type of content, then maybe you want to consider subscribing to

* my curated newsletter [**Short Ruby News**](https://newsletter.shortruby.com/) where I cover weekly Ruby news from all around the internet.
    

<iframe src="https://newsletter.shortruby.com/embed" width="800" height="320" style="border:1px solid #EEE"></iframe>

* or to my personal newsletter below