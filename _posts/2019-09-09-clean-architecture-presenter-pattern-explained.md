---
layout: post
title:  "Clean Architecture Presenter Pattern: Explained"
date: 2019-09-07T06:12:52+02:00
author: ivanpaulovich
categories: [ cleanarchitecture, presenter ]
image: assets/images/norman-tsui-PoF0kgQm9hE-unsplash.jpg
featured: true
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

1. The first task is to move the exceptions handling to a Filter. It will catch exceptions globally, so methods at the top level should not implement `BadRequest` handling.
2. Where should go the Excel file creation? Create a Presenter.

The presenter will expose methods 

![Cluttered Controller]({{ site.baseurl }}/img/presenter/clean-architecture-controller-2.png)
