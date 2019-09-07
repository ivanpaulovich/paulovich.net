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

Wrong! Validation is the most common source of issues in Web applications. Let me explain the issues.

* **Untestable Validation Code:** To test the validation logic it is required to write tests against the whole universe of combinations, it is not pratical.
* **Mixed Validation Code:** When every method is concerned about input validation.
* **Business Logic depedent on Frameworks:** 

