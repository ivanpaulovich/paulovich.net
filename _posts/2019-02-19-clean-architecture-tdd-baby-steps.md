---
layout: post
title:  "Clean Architecture and TDD"
date: 2019-02-19T06:12:52+02:00
author: ivanpaulovich
categories: [ TDD ]
image: assets/images/17.jpg
---
Nowadays all software development companies are self-titled Agile (if you are not Agile you are not cool right?). Most companies are following the SCRUM ceremonies, they adopted small developers teams, they have a SM and PO roles. 

> Are SCRUM ceremonies enough to be Agile? How the software implementation interfer? 

I could say a lot about a company on how they answer the following question: 

> Are teams delivering working software to real users on every iteration (including the first) and gathering feedback?

If the answer is *No* I suppose the following:

* Teams usually do not deliver on the first iteration because they are designing an architecture or adding framework dependencies. 
* They do not gather user feedback on every iteration.
* Long lead time for every new feature, the business value is retained for long time.

Agile is about collaboration with people, gathering feedback from real users!

> Why software take so long to reach the production environment? Why they have so many bugs?

The software architecture is the main reason for features taking long time to be released to production. It is common that teams do a lot of effort designing a big archictecture up front that requires fancy frameworks for every feature. The end result is an application overwhelmed of dependencies, error prone and hard to change.

The application reachs production with many bugs because the team spend most of the time configuring the web server, working with ORM frameworks and the user interfaces. The team did not have time in collaboration with the users trying to understand the use cases and implementing the business rules.

By a lack of confidence, the developers try to implement the frameworks on the initial sprints to avoid getting caught unprepared on the later sprints. This decision create coupling with technology. Let me ask some questions: - Do we need a database server to implement the business rules? Do we need a running web server to gather the real user feedback?

> We don't need a SQL Server or a running Web Server to gather user feedback on the business rules.

To design a tightly coupled architecture we just need to begin with configuring the database, the web server, the frameworks then in the remaining time implementing the business rules.

> With so many moving parts we fail to get the real user feedback! Worse... it will fail slowly.

Now... suppose that we wish to design a software architecture that prioritize collaboration with Domain Experts. We desire an application loose coupled to a database and the web server, we want to decide about these details when we have enough information. Is implementing the business requirements the priority for your organization? If that's the case you will need to work on your programming disciplines.

## Just Enough Architecture

What if we could focus on business requirements and ignore everything else? The idea behind "Ports and Adapters" is to decouple the high level modules from the low level modules, in simple terms you could decouple the business rules from the database and user interface.

![Hexagonal Architecture]({{ site.baseurl }}/img/hexagonal-architecture/hexagonal-architecture.png)

As you can see on the left side there are driving actors:

* Test Harness
* User Interface

The secondary actors are on the right side:

- Mocked Database
- SQL Database Adapter
- Mocked Web server
- Web server Adapter

The use cases are implemented inside the **Application Layer**.

What I am saying is that whatever the right or left side dependencies are you always can delay their implementation by prioritizing tests and mocks. The use cases are the important thing you need to focus on! Is there a correct order to implement an Hexagonal Architecture?

## Ports and Adapters Implementation Workflow

The benefit of "Ports and Adapters" is that the application use cases could be implemented in isolation from external services, so we can delay the database and web server implementation by creating fake implementations. 

> What about the driving actors? When should I implement them?

![First Step]({{ site.baseurl }}/img/hexagonal-architecture/guided-by-tests-1.png)

The **first driving adapter** you should implement are the **Test Harness**. And to run tests you don't need an user inteface, see how you don't need to worry about button colors and font faces? These tests will guide the use case implementation against a mocked database.

![Second Step]({{ site.baseurl }}/img/hexagonal-architecture/guided-by-tests-2.png)

With the knowledge acquired by the unit tests implementation you can more confident design the **User Interface** then get user feedback. Every stage is a learning process, be open to change the use cases implementation and test harness at anytime!

![Third Step]({{ site.baseurl }}/img/hexagonal-architecture/guided-by-tests-3.png)

You now can go deeper in details and implement how the application consume the database, and you can run your existing tests against this secondary actor. Should I say that you will do small changes in the application use cases to support this new adapter? You will!

![Final Step]({{ site.baseurl }}/img/hexagonal-architecture/guided-by-tests-4.png)

The last step you run the **User Interface** against a real database implementation and get more feedback!

### Optional Acceptance Tests

We could create tests for the User Interface. Considering that you followed the previous steps.

## Why TDD is Agile?

Agile methodology is not about doing things quickly without quality. When designing tests you may feel that you are wasting time and in reality is the opposite:

> The only way to go fast is to go well. Every time you yeild to the temptation to trade quality for speed, you slow down. Every time. Uncle Bob.

Software should be implemented incrementally and on every sprint you should acquire business knowledge that help you be effective on the next sprint.