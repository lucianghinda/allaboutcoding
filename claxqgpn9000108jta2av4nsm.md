# Write tests on all test levels

When thinking about testing, consider the following points: 

**You are not writing tests only for yourself** but also for other developers who might join your project. They might come later, and I need to understand the code from a unit/component/object/method perspective, not only from an integration perspective. 

**Write unit and integration tests to uncover common scenarios and edge cases**. So that when someone wants to understand some edge cases in the code, they will have fast feedback or can use the tests to document how the code works and why it works that way. This is because tests have context: pre-conditions, input on one side, and expected result and post-conditions. Thus together with the code, the code comments are an integral part of describing a piece of functionality. 

I often hear the idea: Don't write unit tests! They are useless! You have to change them too often when you change the code! I will not go into what good unit tests are or the difference between functional and non-functional/structural unit testing, but only talk about the idea of having them. 

**The worst possible scenario is a collaborative project where the team members feel there are not enough tests or that tests are not covering the code correctly. This creates fear of changing the code. **Thus, the code quality will decrease. 

Imagine a scenario where you write a lot of integration tests or system tests that exercise a lot of scenarios so that you consider to have good coverage. Now you want to change a line in a method: 
- What tests do you run to have quick feedback about this change? 
- Are the ones you choose covering well the cases of how this method should behave? Or did you missed to run one integration test? 
- Do you pay the price to run the entire full suite for just changing a line of code in a method? 

**There is a concept called Test Levels which is old wisdom learned through a lot of practice. ** Before adopting a hard stance against any test level, it would be good advice to understand what they are and, most importantly, to understand what category of bugs each test level can uncover.  Suffice it to say that the tests discovered at one level are usually different than the bugs discovered at another level. It might be that some experts in your field are advocating for not writing unit tests, and maybe for the projects, they are right. 

**If you work alone on your project, hands off, do as you like.** Tests or no tests, either way, is a good choice _for you_. It is a good choice because you support the consequences. Thus, you are entitled to choose to write tests only on the integration level or even not to write any tests. 

**When working in a team, please add tests in all test levels: unit, integration, system, and acceptance if your project allows this.**

Don't worry about too many tests, 
> “Or worry, but know that worrying is as effective as trying to solve an algebra equation by chewing Bubble gum”

Your colleagues will delete the tests that do not make sense or need to understand why a test fails, thus having **an excellent opportunity to reflect at Chesterton’s Fence and answer why the code is there.**

Yes, they might spend time understanding tests and then decide after a refactoring/change that it is irrelevant and delete it. This is not a waste of time. This is good, this is learning, and it helps improve the system's quality while expanding the understanding of how it works under the hood.

As we are here, let me say this: **don't delegate writing tests for later.** It will most probably not happen: I rarely see a couple of days later a PR saying, “Adding tests for feature X.” 

*Once a feature is implemented, assume you will move to another feature. So the best time to add tests or think about code quality is when you build the feature. You have the context and focus.*

**Testing is just another face of making your code easy to debug.**
If you care about improving code reliability, tests are an essential component. The same goes if you care about making your code easy to understand; consider tests an integral part of making your code easy to read by the next developer. Add tests on the proper test level where you see fit.

When thinking about unit testing/component testing, **if you find writing a test for a piece of code hard, then don't be afraid to change the code. **Most probably, the code is too complex and has too many dependencies. The most common solution is to split the code into smaller components. 

**Please do not write clever code in tests. Write them in the simplest possible way.** No meta-programming, fancy abstractions (except for the test framework), branches, and conditions. 
I dare to say don't DRY your tests. Maybe write some data generators or specific assertions to your domain that can be reused.  







