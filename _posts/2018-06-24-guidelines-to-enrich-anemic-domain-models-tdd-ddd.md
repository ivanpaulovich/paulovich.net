---
layout: post
title:  "Guidelines to Enrich Anemic Domain Models with TDD/DDD"
date: 2018-06-24T06:12:52+02:00
permalink: /guidelines-to-enrich-anemic-domain-models-tdd-ddd/
redirect_from:
  - posts/guidelines-to-enrich-anemic-domain-models-tdd-ddd/
author: ivanpaulovich
categories: [ DDD ]
image: assets/images/17.jpg
---
In my previous blog posts you could see [Clean Architecture](https://paulovich.net/clean-architecture-for-net-applications/) and [Hexagonal](https://paulovich.net/hexagonal-architecture-dot-net/) implementations. Continuing this series I am going deeper on the Domain Layer, through my journey of building rich domain models I had bad and good experiences that now I would like to share with you. These are my opinionated approaches that could streamline your learning process. What are the business rules? The business rules would make or save the business money, irrespective of whether they were implemented on a computer or manually. This kind of rules are simple to be described in words as they do not require a database, in fact the database are just an _IO device that our software requires_ to persist state. We could say the same about the Web, the way we present the information to our users has nothing to do with the business rules. That is the mine mindset but what I find in our industry is a mix of business, persistence and frameworks.

![Photo by li tzuni on Unsplash]({{ site.baseurl }}/img/li-tzuni-507346-unsplash.jpg)

Photo by li tzuni on Unsplash\[/caption\] To begin we need to understand the code issues we want to avoid before decide to invest time and effort on building rich domains models. The code issues I am referring to are known as code smells, and they are associated with architecture and development problems.

Code Smells to Avoid
--------------------

The opposite of the Rich Domain Models are the Anemic Domain Models, in this second one the business logic are implemented far from the classes that own the data, it brings low cohesion and nonexistent encapsulation. The next topics introduce common code smells in Anemic Domain Models.

### Feature Envy

It’s the situation where a client class access the fields of another class more than it's own data. In order to keep the policies of the second class consistent the consumer needs to validate and manipulate multiple fields together. This code smell is easy to find:

*   When "Application Services" or "Extension Methods" are envy of the Entities fields. These kind of classes implement the policies that should be managed by the data owner, in most of cases the Entities classes.

### Primitive Obsession

It’s the use of primitive types (string, int, float, arrays) for simple tasks that ensures the business rules. There common issues are seen:

*   When you see "Security Social Numbers", "Phone numbers" and Money been repeatedly validated from the UI through the database. To fix this issue we need to create a custom type to encapsulate this logic.
*   When a client class needs to manipulate arrays in external classes in order to keep the data and policies consistent. Generic Lists and Collections leaks abstraction, they provide access to built-in methods to manage the items that are not desired by the business rules. To fix this issue we need to create an Adapter class with the proper methods to manipulate the items.

### Abuse of the Public Setters

> When composing objects into a new type, we want the new type to exhibit simpler behavior than all of it's component parts considered together. Steve Freeman (GOOS)

The classes that exposes all the internal complexity by allowing the consumers to change the internal fields at any time and anywhere. Due to the non-existent encapsulation the consumers need to understand how to change the class properties and to keep the state consistent. To fix this issue we need to remove the public setters and move the logic to proper methods.

### Business Classes Designed for ORM

Instead of design the classes to meet business requirements the classes are designed to meet the ORM frameworks requirements. The end result are classes that only reflect tables structure.

### Anemic Classes

It’s the photograph of a poor implemented business requirements. This kind of classes only store data, they do not implement behaviors or policies. These code smells alone doesn't mean that the code is bad at all. In certain conditions these characteristics are necessary. The problem happens when multiple code smells are combined in a single code base, then the software gets harder to change, the regression tests are required for small changes and the bugs are frequent. Check out the [Refactoring Guru](https://refactoring.guru/refactoring/smells) for a compiled list of code smells. Let's build a new mindset, the journey is worth it!

How to Enrich Domain Models?
----------------------------

The reason we invest effort on enrich the Domain is to prove it's viability, we can do a lot of work without worrying about the database or presentations concerns.

![Photo by Victor Freitas on Unsplash]({{ site.baseurl }}/img/victor-freitas-593843-unsplash.jpg)

To design a rich model we need to concern only on business policies, all the external details like Databases, HTTP and serialization will be addressed later. In our example, we define the business with the following use cases and requirements:

1.  The customer can register a new account.
2.  Allow to deposit into an existing account.
3.  Allow to withdraw from an existing account.
4.  Accounts can be closed only if they have zero balance.
5.  Accounts does not allow to withdraw more than the current account balance.
6.  Allow to get the account details.
7.  Allow to get the customer details.
8.  It's required from the customer to fill Name, SSN and to deposit an initial amount when registering.

We are going straight to the entities and use cases and see what we can do with OO principles to design a Rich Domain Model. We could identify the following patterns:

*   **Aggregate Roots:** Customer and Account
*   **Entities:** Credit and Debit
*   **Value Objects:** Name, SSN and Amount
*   **Use Cases**: Register, Deposit, Withdraw, Close, Get Customer Details, Get Account Details.

We alert that our model are persistent ignorant, it privileges the business and we avoid ORM frameworks interference in our classes. To design the Customer we think first on the test specification. We would like the Customer API to be used this way:

#### CustomerTests.cs

We point out that the Customer and Account are aggregate roots and they must know each other by their IDs. The Customer.Register(..) method does not accept the Account instance, instead accepts only the AccountId.

<script src="https://gist.github.com/ivanpaulovich/79d405a602685bb2e8468aa6dd00f42b.js"></script>

#### Customer.cs

All fields are private sets so all the state changes are made by the methods, the specific Accounts property return an IReadOnlyCollection to prevent unexpected changes from consumers. In this class the state consistency are ensured from the constructor that requires the customer details to the Register(..) method. Previously, I said that I would not corrupt the Model in order to persist the state. I made and exception for the factory method that receives the complete Customer fields as parameters and it creates a Customer instance. To persist the objects the repository can use the public properties to get the Customer state.

<script src="https://gist.github.com/ivanpaulovich/5d3f702a55a4700dd23a272a2dca5617.js"></script>

#### Account.cs

I added the sealed modifier to the Account class to prevent inheritance. I am an advocate of composition over inheritance, and I added this modifier to the domain classes to be transparent with my intention. I don't want the consumers creating unnecessary coupling. The transaction history can be changed only in the next situations:

*   By the deposit method which adds and transaction.
*   By the withdraw method which adds and transaction.
*   By the factory method which recreates the list.

The consistency is ensured by not allowing the client to make changes on the TransactionCollection property.

<script src="https://gist.github.com/ivanpaulovich/21ca4c7b445764adcfc676c503a13348.js"></script>

#### SSN.cs

This class is a value object for the Swedish Personnummer and it encapsulates the complexity of validating the string format. Whenever I am refer to a string personnummer I can use this class.

<script src="https://gist.github.com/ivanpaulovich/6c7776aaff93e29e21ec3e037c9df2e9.js"></script>

Source Code
-----------

There are more examples of Rich Domain in my GitHub repository. You can find the Aggregates, Entities and the Values objects. Also everything is covered by Unit Tests. You can download the source code on [DDD/TDD Rich Domain](https://github.com/ivanpaulovich/ddd-tdd-rich-domain).

```
git clone https://github.com/ivanpaulovich/ddd-tdd-rich-domain.git
cd ddd-tdd-rich-domain
./build.sh
```

<span data-mce-type="bookmark" style="display: inline-block; width: 0px; overflow: hidden; line-height: 0;" class="mce\_SELRES\_start">﻿</span>

Conclusion
----------

Building rich domains is not an easy task, in fact it requires much more to think on implementing the business requirements and how to hide the internal details. Fortunately we can leverage on TDD practices to validate the API usage, and to ensure it's correctness. The DDD patterns help us understand how the components should work together. We highlight that the principles of high cohesion and low coupling are required to lower the complexity of the code base. What do you think?