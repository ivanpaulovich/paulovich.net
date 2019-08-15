---
layout: post
title:  "Clean Architecture Essentials - Part I"
date: 2019-04-20T06:12:52+02:00
author: ivanpaulovich
categories: [ CleanArchitecture ]
image: assets/images/17.jpg
---
I have been following the Clean Architecture principles on the [todo](https://github.com/ivanpaulovich/todo) development and I would like to share my experience with you. The strategies and decisions I have made could help you in future projects.

## Use Cases

The first step I did was to pick a simple domain, I chose the "Tasks Management" domain and the expected use cases are listed below:

* Add a new task.
* List all tasks.
* Rename task title.
* Mark task to done.
* Mark task to incomplete.
* Remove task.

To summarize the Clean Architecture style I would say that it is an application designed around use cases with the implementation guided by tests. So the next step in the `todo tool` development was to setup the test project.

I created the `TodoList.UnitTests` for tests and `TodoList.Core` for the use cases implementation. That is right, when I started the development I did not create an "Web API project" instead I consider that the first application consumer are the unit tests.

## Tests

Going further. The tests and production code are created at the same time. For instance when designing the TodoUseCase, I created a test case that verifies null input.

This simple test guides me into discovering the classes that I need.

```
[Fact]
public void GivenNullInput_ThrowsException()
{
    var sut = new TodoUseCase();
    ...
}
```

The next thing I did was to create the TodoUseCase class definition:

```
public sealed class TodoUseCase
{
    public void Execute(Request request)
    {
        throw new NotImplementedException();
    }
}
```

Then I continue with the test design:

```
[Fact]
public void GivenNullInput_ThrowsException()
{
    var sut = new TodoUseCase();
    Assert.Throws<Exception>(() => sut.Execute(null));
}
```

Later I go back to the production code:

```
public sealed class TodoUseCase
{
    public void Execute(Request request)
    {
        if (request == null)
            throw new Exception("Request is null");

        ...
    }
}
```

This is the beginning of Clean Architecture series. What do you think?



