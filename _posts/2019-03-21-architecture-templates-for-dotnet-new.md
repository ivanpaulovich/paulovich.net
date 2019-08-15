---
layout: post
title:  "Architecture Templates for dotnet new"
date: 2018-05-22T06:12:52+02:00
categories: jekyll update
permalink: /check-out-these-awesome-hexagonal-and-clean-architectures-implementations/
redirect_from:
  - /posts/check-out-these-awesome-hexagonal-and-clean-architectures-implementations/
author: ivanpaulovich
categories: [ cleanarchitecture ]
image: assets/images/17.jpg
---
I am releasing an new version of my [Architecture Templates for dotnet new](https://dotnetnew.azurewebsites.net/pack/Paulovich.Caju). We are working on testing, compatibility and documentation.

Paulovich.Caju 0.4.0 Release notes
----------------------------------

-   New architecture tips for each layer in Clean Architecture template. Check out the [blog post](https://paulovich.net/clean-architecture-for-net-applications/).
-   Command line breaking changes. See the topic below.

How to install the latest version
---------------------------------

To install the latest version use:

```
dotnet new -i Paulovich.Caju
```

Then run **dotnet new** and check the templates list:

| Templates | Short Name | Language | Tags |
|-----------|------------|----------|------|
| Hexagonal Architecture for .NET Applications! | hexagonal | [C#] | Common/Library/Web API |
| Event-Sourcing for .NET Applications! | eventsourcing | [C#] | Common/Library/Web API |
| Clean Architecture for .NET Applications! | clean | [C#] | Common/Library/Web API |

Command Line Changes
--------------------

I changed the short name **caju** to three more specifics commands **hexagonal**, **eventsourcing**and **clean**.

The old commands were:

```
dotnet new caju --architecture-style hexagonal
dotnet new caju --architecture-style eventsourcing
dotnet new caju --architecture-style clean
```

The **new commands** in Paulovich.Caju 0.4.0 package are:

```
dotnet new hexagonal
dotnet new eventsourcing
dotnet new clean
```

Much more clear now! Remember that each template has slight differences, so add the **--help** for details.

This templates segregation are required to speed up the updates.