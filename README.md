# Restless OfxSharper
Provides extended Ofx services: parsing of multiple statements, closing statements, financial institution profile info; 
and Ofx request building.

## Main Features
- Build an Ofx request for a financial institution profile (what types of online services it supports) 
and transform the response into a usuable object.

- Build an Ofx request for an account statement (or multiple statements) and transform the resulting response. 
Request and response can include account closing statements as well.

## Code Samples
The following code samples demonstrate a few common scenarios.

### Get FI Profile

```c#
public void OfxProfileExample()
{
    // You need to get your own bank details into an IBank object.
    IBank bank = GetMyBankInfo();

    var builder = OfxFactory.CreateBuilder();

    builder.BuildOfxRequest(() =>
    {
        builder.BuildSignOnMessageSet(bank);
        builder.BuildProfileMessageSetRequest();
    });

    // Send request to the bank and get the response
    string response = GetResponseFromBank(builder.RequestText);
    // Transform response string. Ready to go.
    OfxProfileResponse profile = OfxFactory.CreateProfileResponse(response);
    // Do something.
    if (profile.BillPay.CanAddPayee)
    {

    }
}
```

### Bank Statement Download

```c#
public void BankStatementDownload()
{
    // You need to get your own bank details into an IBank object.
    IBank bank = GetMyBankInfo();
    // You need to get your own account details into an IAccount object.
    IAccount account = GetMyAccountInfo();

    OfxRequestBuilder builder = OfxFactory.CreateBuilder();
    builder.BuildOfxRequest(() =>
    {
        builder.BuildSignOnMessageSet(bank);

        builder.BuildBankMessageSet(() =>
        {
            // in your app, you have to determine an appropiate date to start with.
            DateTime fromDate = new DateTime(2017, 1, 1);
            builder.BuildBankStatementRequest(account, fromDate, null, true);
            // add another statement request if you've got another account at the same bank
        });
    });

    // Send request to the bank and get the response
    string response = GetResponseFromBank(builder.RequestText);

    // Transform response string. Ready to go.
    OfxResponse ofx = OfxFactory.CreateResponse(response);
}
```

### Credit Card Statement Download (with closing statements)

```c#
public void CreditCardStatementDownload()
{
    // You need to get your own bank details into an IBank object.
    IBank bank = GetMyBankInfo();
    // You need to get your own account details into an IAccount object.
    IAccount account = GetMyAccountInfo();

    OfxRequestBuilder builder = OfxFactory.CreateBuilder();
    builder.BuildOfxRequest(() =>
    {
        builder.BuildSignOnMessageSet(bank);

        builder.BuildCreditCardMessageSet(() =>
        {
            // in your app, you have to determine an appropiate date to start with
            DateTime dateStart = new DateTime(2017, 1, 1);
            builder.BuildCreditCardStatementRequest(account, dateStart, null, true);
            // Not using a start date or and end date. Bank will deliver whatever they've got.
            builder.BuildCreditCardClosingStatementRequest(account, null, null);
        });
    });

    // Send request to the bank and get the response
    string response = GetResponseFromBank(builder.RequestText);

    // Transform response string. Ready to go.
    OfxResponse ofx = OfxFactory.CreateResponse(response);
    if (ofx.Statements.Count == 1)
    {
        foreach (var closingPeriod in ((CreditCardStatement)ofx.Statements[0]).Closing.Periods)
        {
            // do something
        }
    }
}
```

### Bank and Credit Card Statement Download

```c#
public void BankAndCreditCardStatementDownload()
{
    // You need to get your own bank details into  an IBank object.
    IBank bank = GetMyBankInfo();
    // You need to get your own account details into an IAccount object.
    IAccount bankAccount = GetMyAccountInfo();
    IAccount creditCardAccount = GetMyAccountInfo();

    OfxRequestBuilder builder = OfxFactory.CreateBuilder();
    builder.BuildOfxRequest(() =>
    {
        builder.BuildSignOnMessageSet(bank);

        builder.BuildBankMessageSet(() =>
        {
            // in your app, you'd have to determine an appropiate date to start with.
            DateTime fromDate = new DateTime(2017, 1, 1);
            builder.BuildBankStatementRequest(bankAccount, fromDate, null, true);
        });

        builder.BuildCreditCardMessageSet(() =>
        {
            // in your app, you'd have to determine an appropiate date to start with
            DateTime dateStart = new DateTime(2017, 1, 1);
            builder.BuildCreditCardStatementRequest(creditCardAccount, dateStart, null, true);
            // Not using a start date or and end date. Bank will deliver whatever they've got.
            builder.BuildCreditCardClosingStatementRequest(creditCardAccount, null, null);
        });
    });

    // Send request to the bank and get the response
    string response = GetResponseFromBank(builder.RequestText);

    // Transform response string. Ready to go.
    OfxResponse ofx = OfxFactory.CreateResponse(response);
    foreach (var statement in ofx.Statements)
    {
        if (statement.StatementType == StatementType.Bank)
        {
            // do something
        }
        if (statement.StatementType == StatementType.CreditCard)
        {
            // do something
        }
    }
}
```
