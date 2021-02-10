---
layout: post
title: Test the number of the records in the SQL database with QAToolKit
excerpt_separator: <!--more-->
author: Miha J.
tags: c# tool qatoolkit database sql
---

In part 3 of the [QAToolKit Database library](https://github.com/qatoolkit/qatoolkit-engine-database-net), we will look at asserting the number of records in a particular SQL database table. Now, why would you do that? There are many reasons.

Imagine you have a webshop, and you have the product categories in a table. You want to add a new category for home vacuum cleaners. After the database migration, you want to check if the number of categories in the table is N+1. This way, you can have some assurance that migration and database seeding completed successfully. But why not expand this and run it continuously and get notified when the test fails? It can be a great way to detect anomalies in your database.

Let's test if your `categories` table in our SQL Server database has 100 records in it.

```csharp
var generator = new SqlServerTestGenerator(options =>
{
    options.AddDatabaseRecordsCountRule(new List<DatabaseRecordCountRule>() 
    {
        new DatabaseRecordCountRule() 
        {
            TableName = "categories", 
            Count = 100,
            Operator = "=" 
        } 
    });
});

List<DatabaseTest> scripts = await generator.Generate();
```

After we generate scripts, we can run them and get back results.

```csharp
var runner = new SqlServerTestRunner(scripts, options =>
{
    options.AddSQLServerConnection("server=localhost;user=user;password=mypassword;Initial Catalog=myDatabase");
});

List<DatabaseTestResult> results = await runner.Run();
```

If we serialize the `results` object to JSON we get:

```json
[
  {
    "DatabaseResult": true,
    "Variable": "categories",
    "Script": "IF EXISTS (SELECT * FROM [categories] WHERE (SELECT COUNT(*) AS [count] FROM [categories]) = 100) BEGIN Select 1 END ELSE BEGIN Select 0 END;",
    "DatabaseTestType": "RecordCount",
    "DatabaseKind": "SQLServer"
  }
]
```

`DatabaseResult` is **true**, which means the SQL script returned 1 - categories table do indeed contain 100 records. Additionally the runner returns `DatabaseTestType` and `DatabaseKind` parameters.

You can assert results by xUnit or NUnit, or you can process the results however you want.

Read more about the library [here](https://github.com/qatoolkit/qatoolkit-engine-database-net).