---
layout: post
title:  "Hexagonal Architecture with .NET"
date: 2018-05-26T06:12:52+02:00
permalink: /hexagonal-architecture-dot-net/
redirect_from:
  - /posts/hexagonal-architecture-dot-net/
---
The feedback of the [Clean Architecture for .NET Applications](https://paulovich.net/clean-architecture-for-net-applications/) made me feel like I needed to take a step back and introduce some concepts first, so I will present my implementation of Hexagonal Architecture to make easier to understand them completely. Unfortunately in my previous experiences in different companies, remain very common that applications are built on top of frameworks and databases. I see that many developers are used to implement software that mimics the data tables instead of design software driven by the business domain. As time goes by, the software becomes highly coupled to these external details and what happens next is the application evolution been dictated by what the vendors support. Hexagonal Architecture (aka Ports and Adapters) is one strategy to decouple the use cases from the external details. It was coined by [Alistar Cockburn](http://alistair.cockburn.us/Hexagonal+architecture) more than 13 years ago, and it is getting better with the Onion and Clean Architectures. Let me introduce the Hexagonal Architecture's intent:

> Allow an application to equally be driven by users, programs or tests, and to be developed and tested in isolation from any of its eventual run-time devices and databases.

I need to point out that everything that gives support for external capabilities are just External Details, on the other side everything that contains the Business Rules and Use Cases should be inside the A_pplication Layer_ and they need to be sustained for long time. In our software we have to distinguish between one and the other.

### External Details

In most scenarios we can defer the implementation of external details and still validate the application behavior. If your answer is yes for any of the next questions, you are probably dealing with peripheral details:

*   Does the application needs an database to persist state?
*   Does the application requires an User Interface?
*   Does the application consumes an external API?

These are common external details which can be mocked, faked or their concrete implementation be replaced for different reasons. I suggest to defer their implementation while you discover more of the domain. Keep in mind the Uncle Bob's quote:

> A good architecture allows major decisions to be deferred and a good architect maximize the number of decisions not made.

Visual Studio makes easier to add libraries for Reflection, Serialization, Security and many others Nuget packages in our projects.  The problem begin when we add these libraries to our Application and Domain. These libraries are just details and should be out of the Application. What we can do?

*   Stop writing classes with inheritance from frameworks.
*   Stop going for shinning objects.
*   Focus on the business rules, make them clear on your Application and Domain Layers.
*   Don't fall into tooling traps.
*   Create the appropriate abstraction for these peripheral concerns.

For didactic reasons we call these details as Ports and Adapters in Hexagonal Architecture style.

### Business Rules and Use Cases

The business rules are the fine grained rules, they encapsulate entity fields and conditions. Also the business rules are the use cases that interacts with many entities and services. They together give create process in the application, they should be sustained for a long time. If it's still unclear, this Uncle Bob phrase might help:

> Business Rules would make or save the business money, irrespective of whether they were implemented on a computer. They would make or save money even if they were executed manually.

In the DDD age, we have patterns to describe business with Entities, Value Types, Aggregates, Domain Services and so on. They are a perfect match with Hexagonal Architecture. Moving on, there are design principles that must be clear before implementing Hexagonal Architecture style.

### Dependency Inversion Principle (DIP)

In the next example, the [DIP](https://drive.google.com/file/d/0BwhCYaYDn8EgMjdlMWIzNGUtZTQ0NC00ZjQ5LTkwYzQtZjRhMDRlNTQ3ZGMz/view) was applied when decoupling our Application Services from the Repositories. And this principle was used to decouple many other things in our Application. Let's remember the DIP and navigate through one example:  

> High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details. Details should depend on abstractions.

For the following example checkout the concepts:

*   The DepositService is the **High-level module** that do not depend on database details, instead it depends on IAccountRepository abstraction.
*   The IAccountRepository is the **abstraction** that do not depend on database details.
*   The AccountSQLRepository is the **low-level module** that depends on IAccountRepository abstraction.

To clarify the idea I created the next picture with the before and after applying DIP:

<img class="img-fluid" src="/img/DIP-1-2.png" alt="DIP">
<span class="caption text-muted">DIP</span>

*   On the left side of the next picture we find in blue an Layered Application where the DepositService depends on AccountSQLRepository.
*   And on the right side in green, by adding an IAccountRepository and applying DIP then the AccountSQLRepository has your dependency pointing inwards. aaaaa

The following listing of DepositService shows an implementation. Suggestion: clone the [Acerola repository](https://github.com/ivanpaulovich/acerola-hexagonal-architecture) for the full implementation.

```
public class DepositService : IDepositService
{
  private readonly IAccountReadOnlyRepository accountReadOnlyRepository;
  private readonly IAccountWriteOnlyRepository accountWriteOnlyRepository;
  private readonly IResultConverter resultConverter;

  public DepositService(
    IAccountReadOnlyRepository accountReadOnlyRepository,
    IAccountWriteOnlyRepository accountWriteOnlyRepository,
    IResultConverter resultConverter)
  {
    this.accountReadOnlyRepository = accountReadOnlyRepository;
    this.accountWriteOnlyRepository = accountWriteOnlyRepository;
    this.resultConverter = resultConverter;
  }

  public async Task<DepositResult> Process(DepositCommand command)
  {
    Account account = await accountReadOnlyRepository.Get(command.AccountId);
    if (account == null)
      throw new AccountNotFoundException(
      $"The account {command.AccountId} does not exists or is already closed.");

    Credit credit = new Credit(account.Id, command.Amount);
    account.Deposit(credit);

    await accountWriteOnlyRepository.Update(account, credit);

    TransactionResult transactionResult = resultConverter.Map(credit);
    DepositResult result = new DepositResult(
      transactionResult,
      account.GetCurrentBalance().Value);

    return result;
  }
}
```

That is the main idea behind Hexagonal Architecture, every time our application requires an external service we implement adapter behind an abstraction.

### **Separation of Concerns** (SoC)

Our application requires some external capabilities but the application is not concerned about their implementation details, only their abstractions are visible to the application layer. We apply [SoC](https://en.wikipedia.org/wiki/Separation_of_concerns) by creating boundaries around the Adapters and by allowing them to be developed and tested in isolation. Usually, we have different packages for each Adapter. We could have an specific Adapter for an SQL Database and an specific Adapter for Azure Storage which could be replaced with little effort. That is the idea behind the Hexagonal Architecture, **keep the options open as long as possible and the ability to rollback if necessary**. We can quote Uncle Bob's Plugin Architecture, with the relationship between Visual Studio and Resharper. Not a single line of VS knows about Resharper, but Resharper is developed based on the Visual Studio abstractions. They are developed by different companies one in Seattle and another in Moscow and still running well together.

Hexagonal Architecture Style Characteristics
--------------------------------------------

With this style we have:

*   An independent Business Domain to embody the small set of critical business rules.
*   Application Services to implement the use cases.
*   Ports to get the input.
*   Adapters providing implementations of frameworks and access to databases.
*   Externally the user, other systems and services.

One way to explain the Hexagonal Architecture is by its shapes. Take a look at the following picture:

![](/img/hexagonal-1.png)  

*   The blue potato shape at the center is the Domain and there are reasons for it. Every business domain has its own rules, different specifications from each other, that is the reason of its undefined shape. For instance, I designed our Domain Layer with DDD Patterns.
*   The application has an hexagonal shape because each of its sides has specifics protocols, in our example we have **Commands** and **Queries** giving access to the Application.
*   The Ports and Adapters are implemented outside of the application as plugins.
*   Externally we have other systems.

The direction of the dependencies goes inwards the center, so the Domain Layer does not know the Application Layer but the Application Layer depends on the Domain, the same rule applies to the outer layers.

### Layers

Let's describe the Dependency Layer Diagram below:

<img class="img-fluid" src="/img/Untitled-Diagram-1.png" alt="Dependency Layer Diagram">
<span class="caption text-muted">Dependency Layer Diagram</span>
-------------------------------------------------------------------------------------------------------------------------------------------------------

*   The domain is totally independent of other layers and frameworks.
*   The application depends on Domain and is independent of frameworks, databases and UI.
*   Adapters provides implementations for the Application needs.
*   The UI depends on Application and loads the Infrastructure by indirection.

We should pay attention that the Infrastructure Layer can have many concerns. I recommend to design the infrastructure in a way you can split it when necessary, particularly when you have distinct adapters with overlapping concerns. It is important to highlight the dashed arrow from the UI Layer to the Infrastructure layer. That is the where **Dependency Injection** is implemented, the concretions are loaded closer to the Main function. And there is a single setting in a external file that decides all the dependencies to be loaded.

### The Application Layer

To make simpler the Application Layer implementation, I split it in two stacks: one for the transactions and other for the queries.

*   When the user sends an Deposit Input, it goes to the transactions stack, it is converted into a Command that goes through the DepositService, uses the Entities to enforce the business rules and the transaction is finally persisted in a database by an adapter.
*   The other stack is tinnier and implemented by the Adapters. It is used only for querying view objects.

With this approach we avoid the degradation of our domain because we don't need to represent every Query Results into Entities.

### Use Cases Components

It is very important to organize the Application Layer with the use case vocabulary. I recommend one folder for each use case, as we have in the following example:

*   Command (DepositCommand.cs)
*   Use case Interface (IDepositService.cs)
*   Use case Implementation (DepositService.cs)
*   Command Result (DepositResult.cs)

With this approach we have an application design that supports new use case implementations with fewer changes in existing code base. This keep the work effort for new use cases implementations constants along the sprints in an Agile methodology.

### Queries

For the Query side, in the Application Layer we have only an small interface. And in the Infrastructure Layer we have the Adapter implementation.

```
public interface IAccountsQueries
{
  Task<AccountResult> GetAccount(Guid id);
}
```

By having an guarantee that the query side does not make changes in state. We can take advantage of better solutions for reading. For instance we can use caching, segregated databases to boost performance and it could be done inside the Adapter.

Ports
-----

A Port is an way an Actor can interact with the Application Layer. The role of the Port is to translate the Actor's input into structures the Application Services can understand. For instance a Port could be an Web Form, an Console App or another system. For this article the Port supports the REST protocol and was implemented using WebApi framework.

```
[Route("api/[controller]")]
public class AccountsController : Controller
{
  private readonly IDepositService depositService;

  public AccountsController(
    IDepositService depositService)
  {
    this.depositService = depositService;
  }

  /// <summary>
  /// Deposit from an account
  /// </summary>
  [HttpPatch("Deposit")]
  public async Task<IActionResult> Deposit(\[FromBody\]DepositRequest request)
  {
    var command = new DepositCommand(
      request.AccountId,
      request.Amount);

    DepositResult depositResult = await depositService.Process(command);

    if (depositResult == null)
    {
      return new NoContentResult();
    }

    Model model = new Model(
      depositResult.Transaction.Amount,
      depositResult.Transaction.Description,
      depositResult.Transaction.TransactionDate,
      depositResult.UpdatedBalance
    );

    return new ObjectResult(model);
  }
}
```

The WebApi has Controllers that do not depends on Application Services implementation, its easy to mock this services.

### Port Components

We segregate Port Components by use cases, for the Deposit use case:

*   Request (DepositRequest.cs)
*   Controller + Action (DepositController.cs)
*   Model (Model.cs)

Source Code
-----------

You can download the source code on [Acerola GitHub repository](https://github.com/ivanpaulovich/acerola-hexagonal-architecture) or through the following commands:

```
dotnet new -i Paulovich.Caju::0.5.0
dotnet new hexagonal \
  --data-access entityframework \
  --use-cases full \
  --user-interface webapi
```

Conclusion
----------

With Hexagonal Architecture you design a decoupled software that allows major decisions to be made in the future, all business rules will be isolated from peripheral concerns. And you have the option to try different ports and adapters with less effort. What comes next? Leave your comment.