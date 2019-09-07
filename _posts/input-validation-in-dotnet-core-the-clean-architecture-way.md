---
layout: post
title:  "Input Validation in .NET Core: The Clean Architecture way"
date: 2019-04-20T06:12:52+02:00
author: ivanpaulovich
categories: [ cleanarchitecture, validation ]
image: assets/images/17.jpg
draft: true
---
Almost every software requires some input validation implementation. Due to the importance there are frameworks and guidelines to help us complete the task, should not be dificult to write good validation code right?

Wrong! Validation is the most common source of issues in Web applications. Let me explain the code smells:

* **Untestable Validation Code:** To test the validation logic it is required to write tests against the whole universe of combinations, it is not pratical.
* **Mixed Validation Code:** Every method is concerned about input validation. Complexity increases on every new feature added.
* **Business Logic depedent on Frameworks:** Too much business code wrote using frameworks. What if we need to change the framework?

## What I suggest?

I suggest that we take leverage of frameworks and still write testable code. I would describe the steps as following:

1. Validate the required fields and types in isolation.
2. Validate the fields values format (e-mail, phone number, personnummer).
3. Validate if the combination of fields are valid.

What if we could use the leverage of frameworks and still have testable code? What if the validation code was cared like first-class objects?
