---
layout: post
title:  "Clean Architecture Presenter Pattern: Explained"
date: 2019-09-07T06:12:52+02:00
author: ivanpaulovich
categories: [ cleanarchitecture, presenter ]
image: assets/images/norman-tsui-PoF0kgQm9hE-unsplash.jpg
draft: true
---
The easiest way to identify a poor designed Web application is by looking at their controllers, usually there is a mix of business rules and View Models.
The following code snippet calls a service to get the account details then the transaction list is exported to a excel spreadsheet. Check it out:

```c#
/// <summary>
/// Get an account details
/// </summary>
[HttpGet("{AccountId}", Name = "GetAccount")]
[ProducesResponseType(StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
[ProducesResponseType(StatusCodes.Status500InternalServerError)]
[SwaggerRequestExample(typeof(GetAccountDetailsRequest), typeof(GetAccountDetailsRequestExample))]
public async Task<IActionResult> Get([FromRoute][Required] GetAccountDetailsRequest request)
{
    try
    {
        var getAccountDetailsInput = new GetAccountDetailsInput(request.AccountId);
        var getAccountDetailsOutput = await _getAccountDetailsUseCase.Execute(getAccountDetailsInput);

        var dataTable = new DataTable();
        dataTable.Columns.Add("Amount", typeof(double));
        dataTable.Columns.Add("Description", typeof(string));
        dataTable.Columns.Add("TransactionDate", typeof(DateTime));

        foreach (var item in getAccountDetailsOutput.Transactions)
        {
            var transaction = new TransactionModel(
                item.Amount,
                item.Description,
                item.TransactionDate);

            dataTable.Rows.Add(new object[] { item.Amount, item.Description, item.TransactionDate });
        }

        byte[] fileContents;

        using (ExcelPackage pck = new ExcelPackage())
        {
            ExcelWorksheet ws = pck.Workbook.Worksheets.Add(getAccountDetailsOutput.AccountId.ToString());
            ws.Cells["A1"].LoadFromDataTable(dataTable, true);
            ws.Row(1).Style.Font.Bold = true;
            ws.Column(3).Style.Numberformat.Format = "dd/MM/yyyy HH:mm";
            fileContents = pck.GetAsByteArray();
        }

        var viewModel = new FileContentResult(fileContents, "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet");
        return viewModel;
    }
    catch(DomainException domainException)
    {
        var problemDetails = new ProblemDetails
        {
            Status = 400,
            Title = "Bad Request",
            Detail = domainException.Message
        };

        return problemDetails;
    }
}
```

The controller receives the `getAccountDetailsOutput`, it creates a DataTable then it generates the Excel file. In case of exceptions it returns a BadRequest response. Isn't too much responsibility for a single method?

![Cluttered Controller]({{ site.baseurl }}/img/presenter/clean-architecture-controller-1.png)

We need to declutter this mess!

## Global Exception Handling with Filters

Actions should not implement `BadRequest` or exception handling. We move the exceptions handling to a Filter (at top level), this object will catch exceptions globally.

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

Second you add this filter at the `ConfigureServices(IServiceCollection services)` method:

```c#
services.AddMvc(options =>
    {
        options.Filters.Add(typeof(BusinessExceptionFilter));
    });
```

## The Excel Presenter

We still have the Excel file creation to declutter. Who should be responsible for it? The answer is a **Presenter** object.

The presenter will expose methods 

![Cluttered Controller]({{ site.baseurl }}/img/presenter/clean-architecture-controller-2.png)

```c#
public sealed class GetAccountDetailsPresenterV2 : IOutputPort
{
    public IActionResult ViewModel { get; private set; }

    public void Error(string message)
    {
        var problemDetails = new ProblemDetails()
        {
            Title = "An error occurred",
            Detail = message
        };

        ViewModel = new BadRequestObjectResult(problemDetails);
    }

    public void Default(GetAccountDetailsOutput getAccountDetailsOutput)
    {
        var dataTable = new DataTable();
        dataTable.Columns.Add("Amount", typeof(double));
        dataTable.Columns.Add("Description", typeof(string));
        dataTable.Columns.Add("TransactionDate", typeof(DateTime));

        foreach (var item in getAccountDetailsOutput.Transactions)
        {
            var transaction = new TransactionModel(
                item.Amount,
                item.Description,
                item.TransactionDate);

            dataTable.Rows.Add(new object[] { item.Amount, item.Description, item.TransactionDate });
        }

        byte[] fileContents;

        using (ExcelPackage pck = new ExcelPackage())
        {
            ExcelWorksheet ws = pck.Workbook.Worksheets.Add(getAccountDetailsOutput.AccountId.ToString());
            ws.Cells["A1"].LoadFromDataTable(dataTable, true);
            ws.Row(1).Style.Font.Bold = true;
            ws.Column(3).Style.Numberformat.Format = "dd/MM/yyyy HH:mm";
            fileContents = pck.GetAsByteArray();
        }

        ViewModel = new FileContentResult(fileContents, "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet");
    }

    public void NotFound(string message)
    {
        ViewModel = new NotFoundObjectResult(message);
    }
}
```

## Passing the Output Message Back

```c#
public async Task Execute(GetAccountDetailsInput input)
{
    IAccount account = await _accountRepository.Get(input.AccountId);

    if (account == null)
    {
        _outputHandler.NotFound($"The account {input.AccountId} does not exist or is not processed yet.");
        return;
    }

    GetAccountDetailsOutput output = new GetAccountDetailsOutput(account);
    _outputHandler.Default(output);
}
```

## Replaceable Presenters

![Cluttered Controller]({{ site.baseurl }}/img/presenter/clean-architecture-controller-3.png)
![Cluttered Controller]({{ site.baseurl }}/img/presenter/clean-architecture-controller-4.png)

## Registering Controllers, Use Cases and Presenters

```c#
services.AddMvc()
    .SetCompatibilityVersion(CompatibilityVersion.Version_2_2)
    .AddControllersAsServices();
```

```c#
services.AddScoped<Manga.WebApi.UseCases.V2.GetAccountDetails.GetAccountDetailsPresenterV2, Manga.WebApi.UseCases.V2.GetAccountDetails.GetAccountDetailsPresenterV2>();

services.AddTransient(ctx =>
    new UseCases.V2.GetAccountDetails.AccountsV2Controller(
        new Application.UseCases.GetAccountDetails(
            ctx.GetRequiredService<Manga.WebApi.UseCases.V2.GetAccountDetails.GetAccountDetailsPresenterV2>(),
            ctx.GetRequiredService<Application.Repositories.IAccountRepository>()
        ),
        ctx.GetRequiredService<Manga.WebApi.UseCases.V2.GetAccountDetails.GetAccountDetailsPresenterV2>()
    )
);
```

```c#
services.AddScoped<Manga.WebApi.UseCases.V1.GetAccountDetails.GetAccountDetailsPresenter, Manga.WebApi.UseCases.V1.GetAccountDetails.GetAccountDetailsPresenter>();
services.AddScoped<Manga.Application.Boundaries.GetAccountDetails.IOutputPort>(x => x.GetRequiredService<Manga.WebApi.UseCases.V1.GetAccountDetails.GetAccountDetailsPresenter>());
```
a
