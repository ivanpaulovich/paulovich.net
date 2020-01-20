---
layout: post
title:  "Clean Architecture Essentials"
date: 2018-01-20T06:12:52+02:00
permalink: /clean-architecture-essentials/
author: ivanpaulovich
categories: [ cleanarchitecture ]
image: assets/images/17.jpg
---

# The "Software Architecture"

We usually see Software Architecture descriptions like "The software architecture is an ASP.NET Web API with Entity Framework Core and SQL Server". This article explains why you should describe software by the use cases instead of layers and the frameworks it uses.

Secondly, I will distill the Clean Architecture Principles.

<iframe width="560" height="315" src="https://www.youtube.com/embed/hZGF6RHrr8o" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Architecture is About Usage

By a quick look at the following blueprint, you can easily guess it is for a church, a theater or a place that people can gather together. Mostly because there is an open space with many benches focused on the same direction, big doors so a large number of people can enter and leave quickly.

![Church Blueprint](https://thepracticaldev.s3.amazonaws.com/i/nauj79ytnncw9o0da1fl.png)

It is not a house blueprint, right?

> The challenge in software design is to scream the use cases in source code in a way that the first look tells us what the software does instead of frameworks that it is made of.

The default approach of software development is to prioritize the frameworks and technology details. An e-commerce website will scream Web at you, Model-View-Controller or any other framework building blocks.

Could we design the software in a different way? Let's introduce Clean Architecture.

## Clean Architecture

The Clean Architecture style aims for a loosely coupled implementation focused on use cases and it is summarized as:

1. It is an architecture style that the Use Cases are the central organizing structure.
1. Follows the Ports and Adapters pattern.
  * The implementation is guided by tests (TDD Outside-In).
  * Decoupled from technology details.
1. Follows lots of principles (Stable Abstractions Principle, Stable Dependencies Principle, SOLID and so on).

### Use Cases

Use Cases are algorithms that interpret the input to generate the output data, their implementation should be closer as possible to the business vocabulary.

When talking about a use case, it does not matter if it a Mobile or a Desktop application, use cases are delivery independent. The most important about use cases is how they interact with the actors.

* Primary actors initiate a use case. They can be the End User, another system or a clock.
* Secondary actors are affected by use cases.

A set of use cases is used to describe software. Following the Customer primary actor on the left side, in the middle the Ticket Terminal system and the secondary actors on the right side:

![Ticket Terminal Use Cases](https://thepracticaldev.s3.amazonaws.com/i/q2twal2uljc0qoy7j4kl.png)

This article uses code snippets from sample applications and talks. If you are familiar with .NET, these GitHub projects host the full implementation:

* :globe_with_meridians: [Clean Architecture Manga](https://github.com/ivanpaulovich/clean-architecture-manga).
* :globe_with_meridians: [Todo](https://github.com/ivanpaulovich/todo).

Following the project structure:

![Structure](https://thepracticaldev.s3.amazonaws.com/i/w7todun7jck4lln2nopf.png)

Following the Register Use Case implementation:

```c#
public sealed class Register : IUseCase
{
    public async Task Execute(RegisterInput input)
    {
        var customer = _entityFactory.NewCustomer(input.SSN, input.Name);
        var account = _entityFactory.NewAccount(customer);

        var credit = account.Deposit(_entityFactory, input.InitialAmount);
        customer.Register(account);

        await _customerRepository.Add(customer);
        await _accountRepository.Add(account, credit);
        await _unitOfWork.Save();

        var output = new RegisterOutput(customer, account);
        _outputPort.Standard(output);
    }

    // properties and constructor ommited
}
```

The use cases are first-class objects in the application and WebApi layers. By a quick look at the use case names in the source tree, you can guess that the source code is for a Wallet software.

In natural language the **RegisterUseCase** steps are:

* Instantiate a Customer.
* Open up an Account then Deposit an Initial Amount.
* Save the data.
* Write a message to the output port.

### Ports and Adapters a.k.a Hexagonal Architecture

Clean Architecture applies the Separation of Concerns Principle through the Ports and Adapters pattern. This means that the application layer exposes **Ports** (Interfaces) and **Adapters** are implemented in the infrastructure layer.

* **Ports** can be an Input Port or an Output Port. The Input Port is called by the Primary Actors and the Output Ports are invoked by the Use Cases.
* **Adapters** are technology-specific.

![Ports and Adapters](https://thepracticaldev.s3.amazonaws.com/i/4x5rmc160cak8eip39bx.png)

This is the preferred architectural style for microservices. Unfortunately, I see lots of incomplete implementations and source code that do not get the most of the pattern.

The previous picture shows for each dependency two implementations, one Fake (Test Double) implementation, and one Real Implementation. The purpose of it is to make possible to run the software independently of external dependencies.

> A Fake implementation provides an illusion of external dependencies, it has the same capabilities expected from the real implementation and it runs without I/O.

The issue that I see in many codebases is the focus on implementing tests against Mocks which can only run through Unit Tests. A fake could run in the Production environment, can help the developer get feedback and it really pays back the investment. 

Let's start with the Hexagonal Architecture style intent:

> Develop, test and run an application in isolation of external devices. Allows a developer to get feedback after every new implementation.

Follow [TDD Outside-in](https://paulovich.net/clean-architecture-tdd-baby-steps/) in order to achieve the intent:

1. Start the development from the Test Cases, implement the Use Cases.
2. When you find a dependency, instead of implementing the real one start by creating a Fake (Test Double).
3. Get Feedback, make possible to run your application against the Fakes. You can even publish it to production.
4. Implement the real adapter in isolation.
5. The last step is to create the User Interface.

### Principles

Clean Architecture is full of principles, let's analyze code snippets for the different levels of Stability and Abstraction:

The `IAccountRepository` interface is **highly abstract**, **general** and **stable**. It does not have "implementation", it is a high-level concept and it does not have dependencies.

```c#
public interface IAccountRepository
{
    Task<IAccount> Get(Guid id);
    Task Add(IAccount account, ICredit credit);
    Task Update(IAccount account, ICredit credit);
    Task Update(IAccount account, IDebit debit);
    Task Delete(IAccount account);
}
```

The `AccountRepository` is a **Very Concrete** `sealed class`, and it is
**Very Specific** to Entity Framework and **Unstable** by implementing interfaces and depending on libraries.

```c#
public sealed class AccountRepository : IAccountRepository
{
    private readonly MangaContext _context;

    public AccountRepository(MangaContext context)
    {
        _context = context ??
            throw new ArgumentNullException(nameof(context));
    }

    public async Task Add(IAccount account, ICredit credit)
    {
        await _context.Accounts.AddAsync((EntityFrameworkDataAccess.Account) account);
        await _context.Credits.AddAsync((EntityFrameworkDataAccess.Credit) credit);
    }

    public async Task Delete(IAccount account)
    {
        string deleteSQL =
            @"DELETE FROM Credit WHERE AccountId = @Id;
                    DELETE FROM Debit WHERE AccountId = @Id;
                    DELETE FROM Account WHERE Id = @Id;";

        var id = new SqlParameter("@Id", account.Id);

        int affectedRows = await _context.Database.ExecuteSqlRawAsync(
            deleteSQL, id);
    }

    public async Task<IAccount> Get(Guid id)
    {
        Infrastructure.EntityFrameworkDataAccess.Account account = await _context
            .Accounts
            .Where(a => a.Id == id)
            .SingleOrDefaultAsync();

        if (account is null)
            throw new AccountNotFoundException($"The account {id} does not exist or is not processed yet.");

        var credits = _context.Credits
            .Where(e => e.AccountId == id)
            .ToList();

        var debits = _context.Debits
            .Where(e => e.AccountId == id)
            .ToList();

        account.Load(credits, debits);

        return account;
    }

    public async Task Update(IAccount account, ICredit credit)
    {
        await _context.Credits.AddAsync((EntityFrameworkDataAccess.Credit) credit);
    }

    public async Task Update(IAccount account, IDebit debit)
    {
        await _context.Debits.AddAsync((EntityFrameworkDataAccess.Debit) debit);
    }
}
```

The `RegisterRequest` class is **concrete**, by exposing getters and setters it is **inconsistent** and **specific** to the consumer.

```c#
/// <summary>
/// Registration Request
/// </summary>
public sealed class RegisterRequest
{
    /// <summary>
    /// SSN
    /// </summary>
    [Required]
    public string SSN { get; set; }

    /// <summary>
    /// Name
    /// </summary>
    [Required]
    public string Name { get; set; }

    /// <summary>
    /// Initial Amount
    /// </summary>
    [Required]
    public decimal InitialAmount { get; set; }
}
```

The `RegisterInput` class is **concrete**, a little bit **consistent** and **less specific**.

```c#
public sealed class RegisterInput : IUseCaseInput
{
    public SSN SSN { get; }
    public Name Name { get; }
    public PositiveMoney InitialAmount { get; }

    public RegisterInput(
        SSN ssn,
        Name name,
        PositiveMoney initialAmount)
    {
        SSN = ssn;
        Name = name;
        InitialAmount = initialAmount;
    }
}
```

The last one is the `IAccount` interface that is **highly abstract**, **general** and **stable**.

```c#
public interface IAccount : IAggregateRoot
{
    ICredit Deposit(IEntityFactory entityFactory, PositiveMoney amountToDeposit);
    IDebit Withdraw(IEntityFactory entityFactory, PositiveMoney amountToWithdraw);
    bool IsClosingAllowed();
    Money GetCurrentBalance();
}
```

The Clean Architecture Principles will guide you to place objects with certain properties according to the following spectrum:

![Clean Architecture Spectrum](https://thepracticaldev.s3.amazonaws.com/i/6rdi4hyfx0ebhk0eq02t.png)

Another Clean Architecture representation is by concentric circles, where:

* The more inner in the diagram the more the layer is stable and abstract.
* The dependency direction goes inwards the center. 
* Classes that change together are packaged together.

![Clean Architecture Layers](https://thepracticaldev.s3.amazonaws.com/i/oa8pcds17jnbxtw42j5d.png)

Following another complete example showing that:

* The User Interface and the Infrastructure Layers are very unstable and concrete. Highly specific to the devices they are designed for.
* The Core Layer is highly abstract and general. Very stable.

![Order Ticket](https://thepracticaldev.s3.amazonaws.com/i/4hpaw8rmbkvkq4zbzawk.png)

## Plugin Architecture

During the software development, we will inevitably face discussions about what is the best frontend framework, best database or ORM. We should no fall into these never-ending discussions and deffer decisions like those in the earlier stages. Consider the Uncle Bob quote:

> A good architecture allows major decisions to be deferred.
>  A good architect maximizes the number of decisions not made.

One can easily find arguments to say that NoSQL is the best database, another developer can find good arguments to choose SQL Server as the database. 

My answer to this is that either one should be implemented. The best option is to implement the Fake storage and move on with the project. 

![Plugin Architecture](https://thepracticaldev.s3.amazonaws.com/i/m9ietb03vjnhxn4k3x9z.png)

Keep in mind that you should embrace the change because in the future you will find a Cloud service that will be the better option.

## Ports and Adapters in details

![Alt Text](https://thepracticaldev.s3.amazonaws.com/i/pu5uquekwb6pgr74ctye.png)

# Pluggable User Interface

> Decoupling the User Interface is equally important than decoupling Repositories and Services but we usually don't put much effort into it. This practice leads to Controllers that look like God classes, hard to test and to maintain.

Suppose that you have a GetAccountDetailsUseCase. It should display one of the following options:

1. The Account Details.
1. Not Found in case it does not exits.

The initial code should look like:

```c#
public async Task<IActionResult> Get([FromRoute][Required] GetAccountDetailsRequest request)
{
    var input = new GetAccountDetailsInput(request.AccountId);
    try
    {
        var output = await _useCase.Execute(input);
        return Ok(output);
    }
    catch (AccountNotFoundException ex)
    {
        return NotFound(ex.Message);
    }
}
```

I wish that the Controller does not know about the output message to decide which View to return. Let's delegate these responsibility to the Presenter.

![User Interface](https://thepracticaldev.s3.amazonaws.com/i/2f0srp30n1sqdedkeu7p.png)

Both `Controller` and `UseCase` implementations uses the same `Presenter` instance. The Controller does not know about the output message, so we can have an Action that looks like:

```c#
/// <summary>
/// Get an account details
/// </summary>
[HttpGet("{AccountId}", Name = "GetAccount")]
[ProducesResponseType(StatusCodes.Status200OK, Type = typeof(GetAccountDetailsResponse))]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public async Task<IActionResult> Get([FromRoute][Required] GetAccountDetailsRequest request)
{
    var input = new GetAccountDetailsInput(request.AccountId);
    await _useCase.Execute(input);
    return _presenter.ViewModel;
}
```

The Presenter is in charge of translating the Value Objects into a WebApi response. Fortunately the Value Objects exposes a `ToDecimal()` or `ToString()` methods which converts it into primitive types.

The `GetAccountDetailsPresenter` implements both `NotFound` and `Standard` methods mimicking the StdOut and StdErr from Unix. These methods creates the ViewModel object.

```c#
public interface IOutputPort
{
    void NotFound(string message);
    void Standard(GetAccountDetailsOutput getAccountDetailsOutput);
}

public sealed class GetAccountDetailsPresenter : IOutputPort
{
    public IActionResult ViewModel { get; private set; }

    public void NotFound(string message)
    {
        ViewModel = new NotFoundObjectResult(message);
    }

    public void Standard(GetAccountDetailsOutput output)
    {
        var transactions = new List<TransactionModel>();

        foreach (var item in output.Transactions)
        {
            var transaction = new TransactionModel(
                item.Amount.ToMoney().ToDecimal(),
                item.Description,
                item.TransactionDate);

            transactions.Add(transaction);
        }

        var response = new GetAccountDetailsResponse(
            output.AccountId,
            output.CurrentBalance.ToDecimal(),
            transactions);

        ViewModel = new OkObjectResult(response);
    }
}
```

The `GetAccountDetailsUseCase` depends on the `IOutputPort` interface and it calls `NotFound` or `Standard` method accordingly.

```c#
public sealed class GetAccountDetails : IUseCase, IUseCaseV2
{
    private readonly IOutputPort _outputPort;
    private readonly IAccountRepository _accountRepository;

    public GetAccountDetails(
        IOutputPort outputPort,
        IAccountRepository accountRepository)
    {
        _outputPort = outputPort;
        _accountRepository = accountRepository;
    }

    public async Task Execute(GetAccountDetailsInput input)
    {
        IAccount account;

        try
        {
            account = await _accountRepository.Get(input.AccountId);
        }
        catch (AccountNotFoundException ex)
        {
            _outputPort.NotFound(ex.Message);
            return;
        }

        var output = new GetAccountDetailsOutput(account);
        _outputPort.Standard(output);
    }
}
```

## Adding a Mediator

```c#
/// <summary>
/// Get an account details
/// </summary>
[HttpGet("{AccountId}", Name = "GetAccount")]
[ProducesResponseType(StatusCodes.Status200OK, Type = typeof(GetAccountDetailsResponse))]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public async Task<IActionResult> Get([FromRoute][Required] GetAccountDetailsRequest request)
{
    var input = new GetAccountDetailsInput(request.AccountId);
    await _mediator.PublishAsync(input);
    return _presenter.ViewModel;
}
```

You notice that I added the `mediator` instance to decouple the Controller and the `UseCase`, it means that the Controller will produce messages, give them to the mediator then the mediator will deliver the message to the appropriate UseCase. Of course, you can invoke the UseCase directly, verify whats works best for your project.

By definition the Output messages given by the UseCase are a consistent and immutable objects, it uses Value Objects to describe the business state.

# Evolutionary Architecture

All developers will build a Task List app at least once. As a .NET Developer we always start thinking by creating an WebApi first then dig into the business details, for this sample application I wanted to proceed differently.

![Todo Use Cases](https://thepracticaldev.s3.amazonaws.com/i/p0djcfppl8rg337xecvm.png)

I started the development from the Unit Tests, implementing the Use Cases independently and for each dependency creating a Fake. After a while I decided to create a SQL Server database because that is what .NET Developers do, we spin up a SQL Server so we can persist taks ;)

To make it short, at the end a Console UI and a Storage to GitHub gist was good enough for me.

![Evolutionary Architecture](https://thepracticaldev.s3.amazonaws.com/i/16lvcpag4bddy0dcgcih.png)

# Wrapping Up

* Clean Architecture is about usage and the use cases are the central organizing principle.
* Use cases implementation are guided by tests.
* The User Interface and Persistence are designed to fulfill the core needs (not the opposite!).
* Defer decisions by implementing the simplest component first.

The [source code is on GitHub](https://github.com/ivanpaulovich/clean-architecture-manga) and it is updated frequently with new videos and pull-requests. Check it out!
