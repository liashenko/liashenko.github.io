---
layout: post
title:  "Agile: Moving Fast"
date:   2020-04-18 20:00:00 +0300
permalink:   agile
categories: [agile]
tags: [agile, development, tests, refactoring, continuous deployment]
---

**Agile is about moving fast in short cycles.**  
Iterations and small releases to production (a few features at a time) give immense value to both customers and developers.
* Do we build the right thing? 
* Can we adapt to the business changes quickly?
* Are we able to deliver fast?  

**Agile is about conversations and continuous collaboration.**  
Software developers, customers and everyone interested in making the product experience better should be involved in ongoing conversations.  
**Agile is about technical excellence.**  
Software internal quality should be high to be able to move fast and change the direction rapidly.

### Conversations
>Business people and developers must work together daily throughout the project.

* Do we solve the right problem?
* Do we solve the problem in the right way?
* How can we improve?

Developers should be in close contact with experts who understand the business domain. They know the problem in detail, developers know how to code. Talk and together build the best possible product.

A few agile tools to help capturing conversations:
* A **user story** is a software feature described from an end-user perspective. A user story state the type of user, what they want and why.
* **Acceptance tests** are tests that validate how the feature should work. They usually go hand in hand with user stories. Working from concrete acceptance tests helps to understand what’s being asked for.

### Technical excellence
> Welcome changing requirements.  
> Continuous attention to technical excellence and good design enhances agility. 

Technical excellence is the only way to be agile, the only way to be able to adapt to changing requirements rapidly.  
Neglecting internal software quality and building up cruft slows down the development of new features.
<center><img src="/assets/posts/agile/tech_debt.jpg" style="max-width:350px"></center><br/>

To make the internal software quality high, consider the following good practices:
* Automated tests to validate if the software is working as expected.
* Refactor to keep the software design simple and easy to change.
* YAGNI (**You aren’t gonna need it**) - build only things that are important now.
* Continuous delivery/deployment.

#### Automated Testing
Instead of testing manually the same thing over and over every time, automate your tests.  
Automated tests give developers confidence to make code changes and it allows to move fast.  
Automated tests create a feedback loop that informs developers whether the software is working or not. The ideal feedback loop is:
* Fast. Fast tests inform the developer about the bugs faster, which leads to faster fixes.
* Reliable. Tests should indicate the broken software, not the broken test.
* Isolated. Tests should help to fix broken software and find the root cause fast.

**Test Pyramid**  

One key concept of automated testing is **Test Pyramid**. It’s a great representation of how to approach testing your software. 
<center><img src="/assets/posts/agile/testing_pyramid.jpg" style="max-width:550px"></center>
<center>Testing Pyramid (Agile) vs Ice-cream Cone (Traditional)</center><br/>

The more high-level you get the fewer tests you should have:
* Write lots of small and fast unit tests. 
* Write some integration tests.
* Write a very few end-to-end tests.   

**Unit tests**

Unit tests take a small unit of the software (e.g. class, method) and test it in isolation.  
They should not access the network resources, databases, file system, etc.  
Unit tests are incredibly fast (~ms) and reliable. They tend to create the ideal feedback loop.

**Integration tests**  

Unit tests make sure components work in isolation, but they don’t tell you if the components work together.  
Use integration tests to verify the interaction of different components of the system (including databases, filesystems, network calls to other applications). 

**UI tests**  

UI (end-to-end) tests verify the system as a whole from a user perspective.  
End-to-end tests are more flaky by nature and hard to maintain. The good practice is to have the minimum of them. Automate the most important user steps that reflect the core value of your software.  
For example, the core steps for an online store: *search a product, add a product to cart, checkout*.

#### Refactoring
> When you want to make a change, first, make the change easy. (Warning, this may be hard.) Then make the easy change.

Refactoring is about making software design simple to understand and easy to change. 
* Follow Single Responsibility Principle, break your code into short and simple modules.
* Don't Repeat Yourself, avoid duplicating code.
* Make sure your code is testable.

For more about refactoring, check out this [cool resource.](https://refactoring.guru/refactoring){:target="_blank"}

#### YAGNI
> Simplicity - the art of maximizing the amount of work not done--is essential.

YAGNI is about building features that are important right now. Do not waste time on building something in advance. Don’t try to make the software do everything from the beginning. Don’t make your code bloated and complicated if not needed. Martin Fowler has a great mapping of the costs building features in advance:
* Wrong feature - cost of building, cost of carry.
* Right feature, built wrong - cost of repair, cost of carry.
* Right feature - cost of carry.

#### Continuous delivery/deployment
> Our highest priority is to satisfy the customer through early and continuous delivery of valuable software.  
> Deliver working software frequently, from a couple of weeks to a couple of months, with a preference to the shorter timescale.

<br/><center><img src="/assets/posts/agile/cd.jpg" style="max-width:550px"></center>
<center>Continuous delivery/deployment pipeline</center><br/>
Continuous delivery/deployment is a practice when software is developed and deployed into production in short cycles.
Small releases benefit both customers and developers:
* Customers get working features faster.
* Developers can react to customer feedback in real time. If customers submit bug reports or request for features, developers can release the change even in a few hours.  

To release often internal software quality should be high and error rates should be low. That requires technical practices described above:
* Automated testing
* Refactoring
* YAGNI

Furthermore, consider the good practices below to enable continuous deployment.

**Continuous Integration**  

Continuous integration is a practice when developers regularly commit their code changes, after which automated builds and tests are run.  

**Decouple Release from Deploy** 

Use “Feature toggles” to push the code that is not ready for production yet. 
Example of a feature toggle:
```
if (featureService.isEnabled("NewFeature")) {
   // new functionality
} else {
   // old functionality
}
```

**Canary deployment**  

Deploy the new version of the software to a subset of the infrastructure, monitor and rollback if needed.  
Rolling deployment is better than bing bang deployment because it minimizes risks (including software downtime).

### References
1. [Principle behind the Agile Manifesto](https://agilemanifesto.org/principles.html){:target="_blank"}
2. [The State of Agile Software in 2018](https://martinfowler.com/articles/agile-aus-2018.html){:target="_blank"}
3. [Is quality worth the cost?](https://martinfowler.com/articles/is-quality-worth-cost.html){:target="_blank"}
4. [Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html){:target="_blank"}
5. [Refactoring](https://martinfowler.com/books/refactoring.html){:target="_blank"}
6. [YAGNI](https://martinfowler.com/bliki/Yagni.html){:target="_blank"}
7. [Dark Scrum](https://ronjeffries.com/articles/016-09ff/defense/){:target="_blank"}
8. [No more end-to-end tests](https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html){:target="_blank"}