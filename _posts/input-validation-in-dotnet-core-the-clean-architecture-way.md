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

## What I suggest to do?

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

### 2. Validating fields format in the Domain Layer

Fiels data format validation is a business concern. For that reason I want them to be implemented in the Domain layer. My suggestion is that you add a folder for Value Objects with classes like this:

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
