---
layout: post
title:  "todo: My new Clean Architecture implementation"
date: 2019-04-19T06:12:52+02:00
author: ivanpaulovich
categories: [ cleanarchitecture ]
image: assets/images/17.jpg
---
I released a dotnet tool to manage tasks via the comnand line. The tasks are stored in a `secret gist` in your own GitHub account. Checkout a few commands:

![Todo running in the terminal]({{ site.baseurl }}/img/todo-exported.png)

If you have your machine up to date with the [latest .NET Core SDK](https://dotnet.microsoft.com/download/dotnet-core/2.2) and you are interest to see `todo` in action please run:

```
$ dotnet tool install -g todo
```

After that you need to [grant access gist permissions](https://github.com/ivanpaulovich/todo#setup) to the `todo` tool.

In case you are running in a OSX checkout the issue with [zsh: command not found: todo](https://github.com/ivanpaulovich/todo/issues/30). Ask me about it!