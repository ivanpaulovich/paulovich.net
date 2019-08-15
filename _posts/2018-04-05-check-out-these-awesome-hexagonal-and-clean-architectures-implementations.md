---
layout: post
title:  "Check out these awesome Hexagonal and Clean Architectures implementations!"
date: 2018-04-05T06:12:52+02:00
permalink: /check-out-these-awesome-hexagonal-and-clean-architectures-implementations/
redirect_from:
  - /posts/check-out-these-awesome-hexagonal-and-clean-architectures-implementations/
author: ivanpaulovich
categories: [ cleanarchitecture ]
image: assets/images/17.jpg
---
[Filipe Augusto](https://www.linkedin.com/in/filipe-augusto-lima-de-souza-95a16833/) and I have been designing architectures and adapting legacy systems to more sophisticated market standards for a few years. Software Architecture is not a snapshot, it is a living thing and after several proofs of concept in real world systems, we come to some implementations that cover different scenarios.

To illustrate, we published on GitHub three projects with architecture practices for highly testable, framework and database independent softwares**.**

The first one is the [Acerola Project](https://github.com/ivanpaulovich/acerola/), which follows the Hexagonal Architecture (in the center there's a Domain plus the Application and externally Ports and Adapters).

The second one is the [Manga Project](https://github.com/ivanpaulovich/manga/) that goes beyond the Hexagonal Architecture and uses the rules of dependencies and patterns of Clean Architecture. It places User Cases as first-class objects and dependencies must point only inward, toward high-level policies.

The last one is [Amora Project](https://github.com/ivanpaulovich/amora/), an Angular frontend for the previous microservices.

Everything we've published is fresh new and we are in the alpha releases. We're planning to implement and explore new concepts in the future, as Fitness Functions and show by example integrations between few microservices via Broker and Mediator pattern.

These projects were born in the DevOps, Cloud and Container era, they come with CI/CD, TDD, Docker, .NET Core and Azure! Feedback and pull requests are welcome!

Check out the projects. Worth it!