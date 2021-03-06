---
layout: post
title:  "Designing and Testing Input Validation in .NET Core: The Clean Architecture way"
date: 2019-09-07T06:12:52+02:00
author: ivanpaulovich
categories: [ cleanarchitecture, tdd, validation, ddd ]
image: assets/images/norman-tsui-PoF0kgQm9hE-unsplash.jpg
featured: false
---
Almost every software requires some input validation implementation. Due to the importance there are frameworks and guidelines to help us complete the task, should not be difficult to write good validation code right?

Wrong! Validation is the most common source of issues in Web applications. Let me explain the code smells:

* **Untestable Validation Code:** To test the validation logic it is required to write tests against the whole universe of combinations, it is not practical.
* **Mixed Validation Code:** Every method is concerned about input validation. Complexity increases on every new feature added.
* **Business Logic dependent on Frameworks:** Too much business code wrote using frameworks. What if we need to change the framework?

Have you seen bad code like this?

```c#

public void RegisterCustomer(
        string ssn,
        string name,
        double initialAmount)
{
    if (string.IsNullOrWhiteSpace(ssn))
        throw new SSNShouldNotBeEmptyException("The 'ssn' field is required");

    Regex regex = new Regex(RegExForValidation);
    Match match = regex.Match(ssn);

    if (!match.Success)
        throw new InvalidSSNException("Invalid SSN format. Use YYMMDDNNNN.");

    if (string.IsNullOrWhiteSpace(name))
        throw new NameShouldNotBeEmptyException("The 'name' field is required");

    if (value < 0)
            throw new AmountShouldBePositiveException("The 'Amount' should be positive.");

    //
    // other logic implementation omitted
    //
}

```

Read the next topics to learn how to cure this anaemia or navigate to [Guidelines to Enrich Anemic Domain Models with TDD/DDD](https://paulovich.net/guidelines-to-enrich-anemic-domain-models-tdd-ddd/) for more examples.

## What I suggest you to do?

I suggest that we take leverage of frameworks and still write testable code. I would describe the steps as following:

1. Validate the required fields and types in isolation.
2. Validate the fields values format (e-mail, phone number, personnummer).
3. Validate if the combination of fields are valid.

Suppose that a system manages the Customer's Wallet Account. The first use case is `Register a Customer`, let's see how would be the validation steps:

### 1. Validating fields in isolation with Data Annotations

The Web layer is responsible for many things, we do not want to flood the Web with business logic. Let's keep the framework dependencies under control.

So I would add only the `[Required]` attribute here. The benefit is that these attributes are automatically read by the ASP.NET framework for built in validation. The other benefit is that Swagger extensions use this information to generate the API description, it is too many benefits to avoid Data Annotations.

The rule is:

> Keep it simple.

```c#
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
    public double InitialAmount { get; set; }
}
```

The immediate benefit is that Validation is built in ASP.NET Core, completely integrated with Swagger (Open API Specification). 

![Feature Envy Fixed]({{ site.baseurl }}/assets/img/bad-request.png)

### 2. Validating fields format in the Domain Layer

Fields data format validation is a business concern. For that reason I want them to be implemented in the Domain layer. My suggestion is that you add a folder for Value Objects with classes like this:

In Sweden the `Social Security Number` is called `Personnummer` and the format is YYMMDDNNNN. 

* As instance of SSN only exists if it is valid.
* It is immutable (without methods changing the `_text` property.
* It is serializable.
* It is unique by its internal property values.

```c#
public sealed class SSN : IEquatable<SSN>
{
    private string _text;
    const string RegExForValidation = @"^\d{6,8}[-|(\s)]{0,1}\d{4}$";

    private SSN() { }

    public SSN(string text)
    {
        if (string.IsNullOrWhiteSpace(text))
            throw new SSNShouldNotBeEmptyException("The 'SSN' field is required");

        Regex regex = new Regex(RegExForValidation);
        Match match = regex.Match(text);

        if (!match.Success)
            throw new InvalidSSNException("Invalid SSN format. Use YYMMDDNNNN.");

        _text = text;
    }

    public override string ToString()
    {
        return _text;
    }

    public bool Equals(SSN other)
    {
        return this._text == other._text;
    }

    public override bool Equals(object obj)
    {
        if (ReferenceEquals(null, obj))
        {
            return false;
        }

        if (ReferenceEquals(this, obj))
        {
            return true;
        }

        if (obj is string)
        {
            return obj.ToString() == _text;
        }

        return ((SSN) obj)._text == _text;
    }

    public override int GetHashCode()
    {
        unchecked
        {
            int hash = 17;
            hash = hash * 23 + _text.GetHashCode();
            return hash;
        }
    }
}
```

The name just need to be a string. We could change it to be more restrictive.

```c#
public sealed class Name : IEquatable<Name>
{
    private string _text;

    private Name() { }

    public Name(string text)
    {
        if (string.IsNullOrWhiteSpace(text))
            throw new NameShouldNotBeEmptyException("The 'Name' field is required");

        _text = text;
    }

    public override string ToString()
    {
        return _text;
    }

    public override bool Equals(object obj)
    {
        if (ReferenceEquals(null, obj))
        {
            return false;
        }

        if (ReferenceEquals(this, obj))
        {
            return true;
        }

        if (obj is string)
        {
            return obj.ToString() == _text;
        }

        return ((Name) obj)._text == _text;
    }

    public override int GetHashCode()
    {
        unchecked
        {
            int hash = 17;
            hash = hash * 23 + _text.GetHashCode();
            return hash;
        }
    }

    public bool Equals(Name other)
    {
        return this._text == other._text;
    }
}
```

The amount should be positive.

```c#
public sealed class PositiveAmount : IEquatable<PositiveAmount>
{
    private readonly Amount _value;

    private PositiveAmount() { }

    public PositiveAmount(double value)
    {
        if (value < 0)
            throw new AmountShouldBePositiveException("The 'Amount' should be positive.");

        _value = new Amount(value);
    }

    public override bool Equals(object obj)
    {
        if (ReferenceEquals(null, obj))
        {
            return false;
        }

        if (ReferenceEquals(this, obj))
        {
            return true;
        }

        if (obj is double)
        {
            return (double) obj == _value.ToDouble();
        }

        return ((PositiveAmount) obj)._value == _value;
    }

    public Amount ToAmount()
    {
        return _value;
    }

    internal PositiveAmount Add(PositiveAmount positiveAmount)
    {
        return _value.Add(positiveAmount._value);
    }

    public override int GetHashCode()
    {
        unchecked
        {
            int hash = 17;
            hash = hash * 23 + _value.GetHashCode();
            return hash;
        }
    }

    internal Amount Subtract(PositiveAmount positiveAmount)
    {
        return _value.Subtract(positiveAmount._value);
    }

    public bool Equals(PositiveAmount other)
    {
        return this._value == other._value;
    }
}
```

### 3. Validating fields format in the Domain Layer

The use cases accept Input messages and are made of Value Objects.

```c#
public sealed class RegisterInput
{
    public SSN SSN { get; }
    public Name Name { get; }
    public PositiveAmount InitialAmount { get; }

    public RegisterInput(SSN ssn, Name name, PositiveAmount initialAmount)
    {
        if (ssn == null)
        {
            throw new InputValidationException($"{nameof(ssn)} cannot be null.");
        }

        if (name == null)
        {
            throw new InputValidationException($"{nameof(name)} cannot be null.");
        }

        if (initialAmount == null)
        {
            throw new InputValidationException($"{nameof(initialAmount)} cannot be null.");
        }

        SSN = ssn;
        Name = name;
        InitialAmount = initialAmount;
    }
}
```

## How I use it?

In the Web Layer the controller has an action that requires a `RegisterRequest` object, the action is responsible for creating the RegisterInput object then calling the use case.

* The interesting about Request/Response objects is that they are made of serializable objects (eg. `int`, `double` and `string`). 
* The Request/Response are responsible for carrying the data from and back to the User. 
* Of course you can't trust it, you need to create Input objects.

```c#
/// <summary>
/// Register a customer
/// </summary>
/// <response code="200">The registered customer was create successfully.</response>
/// <response code="400">Bad request.</response>
/// <response code="500">Error.</response>
/// <param name="request">The request to register a customer</param>
/// <returns>The newly registered customer</returns>
[HttpPost]
[ProducesResponseType(StatusCodes.Status200OK, Type = typeof(RegisterResponse))]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
[ProducesResponseType(StatusCodes.Status500InternalServerError)]
[SwaggerRequestExample(typeof(RegisterRequest), typeof(GetCustomerDetailsRequestExample))]
public async Task<IActionResult> Post([FromBody][Required] RegisterRequest request)
{
    var input = new RegisterInput(
        new SSN(request.SSN),
        new Name(request.Name),
        new PositiveAmount(request.InitialAmount));

    await _registerUseCase.Execute(input);

    return _presenter.ViewModel;
}
```

The beautiful of this approach is that the use cases can use Input objects and read the Value Objects which are always valid.

```c#
public sealed class Register : IUseCase
{
    // code ommited for simplication

    public async Task Execute(RegisterInput input)
    {
        if (input == null)
        {
            _outputHandler.Error("Input is null.");
            return;
        }

        var customer = _entityFactory.NewCustomer(input.SSN, input.Name);
        var account = _entityFactory.NewAccount(customer);

        ICredit credit = account.Deposit(_entityFactory, input.InitialAmount);
        if (credit == null)
        {
            _outputHandler.Error("An error happened when depositing the amount.");
            return;
        }

        customer.Register(account);

        await _customerRepository.Add(customer);
        await _accountRepository.Add(account, credit);
        await _unitOfWork.Save();

        RegisterOutput output = new RegisterOutput(customer, account);
        _outputHandler.Standard(output);
    }
}
```

Value Objects are powerful, you can use them everywhere in the Application/Domain layers and you know there are valid.

## Throwing and Catching Exceptions

You saw previously that the validation logic is throwing exceptions, so we need to catch them somewhere. I suggest you to add a filter on the Web layer and all Domain Exceptions are returned as `BadRequest` objects.

```c#
public sealed class BusinessExceptionFilter : IExceptionFilter
{
    public void OnException(ExceptionContext context)
    {
        DomainException domainException = context.Exception as DomainException;
        if (domainException != null)
        {
            var problemDetails = new ProblemDetails
            {
                    Status = 400,
                    Title = "Bad Request",
                    Detail = domainException.Message
            };

            context.Result = new BadRequestObjectResult(problemDetails);
            context.Exception = null;
        }
    }
}
```

## Testing

You can test from different perspectives. I would prioritize testing the Input objects from Unit Tests.

```c#
[Fact]
public void GivenNullSSN_InputNotCreated_ThrowsInputValidationException()
{
    var actualEx = Assert.Throws<InputValidationException>(
        () => new RegisterInput(
            null,
            new Name("Ivan"),
            new PositiveAmount(10)
        ));
    Assert.Contains("ssn", actualEx.Message);
}

[Fact]
public void GivenNullName_InputNotCreated_ThrowsInputValidationException()
{
    var actualEx = Assert.Throws<InputValidationException>(
        () => new RegisterInput(
            new SSN("19860817999"),
            null,
            new PositiveAmount(10)
        ));
    Assert.Contains("name", actualEx.Message);
}

[Fact]
public void GivenNullPositiveAmount_InputNotCreated_ThrowsInputValidationException()
{
    var actualEx = Assert.Throws<InputValidationException>(
        () => new RegisterInput(
            new SSN("19860817999"),
            new Name("Ivan"),
            null
        ));
    Assert.Contains("initialAmount", actualEx.Message);
}

[Fact]
public void GivenValidData_InputCreated()
{
    var actual = new RegisterInput(
        new SSN("19860817999"),
        new Name("Ivan"),
        new PositiveAmount(10)
    );
    Assert.NotNull(actual);
}
```

## My Thoughts on Fluent Validation

I see FluentValidation perfect fit for objects validation that I have not control on design. Suppose that I need to consume a old WCF service and the message has constraints on it. 

I would implement a custom FluentValidator on the infrastructure layer.

I would not use it for Value Objects amd Input Messages validation.

## Conclusion

Use the frameworks with cautions, implement the business validation inside the Domain and clean up fat methods replacing primitive objects with proper Value Objects. 

Want to see it in action? The demo is running on Heroku:

[![Clean Architecture Manga Swagger]({{ site.baseurl }}/assets/images/clean-architecture-manga-swagger.png)](https://clean-architecture-manga.herokuapp.com/swagger/index.html)

Happy Codding!

Do you agree/disagree?
