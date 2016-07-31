---
layout: post
title: "3 misconceptions about BDD"
date: 2014-01-15 14:17
comments: true
---

(the same version of this article can also be found at [ThoughtWorks Insights](http://www.thoughtworks.com/insights/blog/3-misconceptions-about-bdd))

BDD has been often misunderstood among developers, QAs and even BAs. We often hear of teams saying that their project is using BDD, but when we check it out, it turns out to be using only a BDD tool for test automation - and not the BDD concepts itself. So in the end, we hear people arguing about the tools, and not about the ideas that inspired the creation of those tools.

The output of that is a bunch of complaints that we see in blogs all over the internet - people that start to reject the whole idea behind BDD, only because they have tried to use a tool without first changing their attitude towards software development.

We commonly hear of these top 3 complaints about BDD:

### #1 The client doesn't care about testing

This is the most common complaint. It makes complete sense to state that, as what really matters for the client is software that suits the client’s purpose. Starting a discussion on testing somehow seems to give business people the green light to tune out their attention. Also the word testing unfortunately bears a negative connotation in the software development community.

But wait a second, we are talking about behavior driven development, and it has nothing to do with testing. Testing is something you can't do until the software exists. Test is verification, and here we are talking about specifications.

BDD is a design activity where you build pieces of functionality incrementally guided by the expected behavior. In BDD we step away from a test-centric perspective and go into a specification-centric perspective, meaning that this complaint was born misplaced.

### #2 The client doesn't want to write the specifications

This is the second most used complaint. Let's break it down.

**“The client must write the specifications by himself”**

Whoever complains about this is assuming that the client is expected to propose a solution to his or her own problem - the very problem that your software was supposed to address.

If the client writes the specifications he won't benefit from something called **cognitive diversity**. Cognitive diversity comes from groups of heterogeneous people working together. For instance, he needs the advice of engineers that know the technical aspects of the problem that he wants to solve. He also needs a QA to spot edge cases that no one else noticed. Else, he may come up with a solution that is much more complex than it needs to be.

It's unfair to complain about something that we, as the development team, are supposed to help our clients with.

**“The client needs to use the tool to write specifications”**

Not really. What the client really needs to do is to provide to the team information about the problem that he wants to solve and together they come up with concrete examples to lead the development process.

### #3 You can achieve the same without a business readable DSL

This argument is commonly found among developers. Most of these developers state that there is no real benefit in having this new “layer” - describing behavior in business readable language - as it only adds complexity and make the test suite slow.

If we take a look at this complaint when working on a Ruby tech stack, this usually implies that instead of using [Cucumber](https://github.com/cucumber/cucumber), you can simply use [Capybara + RSpec](https://github.com/jnicklas/capybara#using-capybara-with-rspec) to achieve the same results with the benefit of having a faster test suite.

That’s like comparing apples with oranges: they are both completely different things.

The benefit of having a business readable DSL - such as feature files written using Cucumber in this case - has benefits beyond the nuts and bolts of development. It's not just about code; it's about improving the all-important communication within the team.

For instance, it's about having a BA collaborating with a developer and a QA, with all of them making improvements on that single file (the feature file) - which is a non-abstract way of presenting ideas for the software. Plus, they will be using their complementary cognitive capabilities to brainstorm together about the best path to transform a specification into fulfilled business needs.

### A successful case using BDD

How complicated would it be for you to explain to a 3 year old child how a bank transaction works? The same challenge applies during software development, as the client domain can be sometimes pretty cloudy and vague to the delivery team.

We need examples to understand. Realistic examples are a great way to communicate, and we use them often without even thinking about it. Working with real world examples help us to communicate better because people will be able to relate to them more easily.

We can easily realize that when we are working on a complex business domain. A good example on that is a previous project we worked on. The company was an investment bank, and as expected in a domain like this, the terminologies were really complicated, making it hard for the developers to communicate with the BAs from the bank.

So, in an effort to communicate better, part of our process was to have a quick call with the BA before playing a card. This BA would then share with the developers, the feature file that he came up with, describing each of the concrete examples that he thought about.

As we didn't have any QAs in this team, it's important to mention that one of the developers in this session had to keep a QA mindset - trying to spot edge cases, improvements to the scenarios, etc. The other developer on the team would properly focus on the technical challenges to implement it, such as make suggestions to move a scenario to a code level specification, which offers a faster feedback. Everyone could also suggest changes to a scenario's step definitions, in order to make it more meaningful to everyone.

The benefit of having it is that the BAs increased their understanding on the technical challenges of the scenarios and the developers had a clearer idea of the business needs and therefore what really needed to be developed. Plus, whenever changes had to be done, having that single piece of information would make everyone on the team in sync about it.

### Down the rabbit hole

To sum up, the main idea behind BDD is that it's driven to prevent communication gaps, that is having everyone in the team communicating more often, better and based on real world examples and not on abstract and imperative requirements.

We hope that this article helps to make people more aware of the benefits of BDD - and make it clear that tools are only something complementary to the full stack agile methodology that it actually is.

If you want to go deeper on the subject, we strongly suggest the following sources:

* "Specification by example" by Gojko Adzic
* "The RSpec Book" by David Chelimsky
* [Dave Astels and Steven Baker on RSpec and Behavior-Driven Development](http://www.infoq.com/interviews/Dave-Astels-and-Steven-Baker)
* [Gojko on BDD: Busting the Myths](http://gojko.net/2012/06/18/bdd-busting-the-myths/)
