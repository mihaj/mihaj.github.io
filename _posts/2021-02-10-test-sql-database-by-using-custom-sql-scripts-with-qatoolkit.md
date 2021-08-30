---
layout: post
title: Test SQL database by using custom SQL scripts with QAToolKit
excerpt_separator: <!--more-->
author: Miha J.
tags: c# tool qatoolkit database sql
---
<!--more-->
In the last part of the [QAToolKit Database library](https://github.com/qatoolkit/qatoolkit-engine-database-net), we will look into writing custom SQL scripts to assert if something is true or not. There is one prerequisite, you need to write an SQL query that will return some results based on the predicate.

For example, simple scripts might be:

```sql
SELECT * FROM [table] WHERE timestamp = '2016-05-31'
```

or

```sql
SELECT * FROM [table] WHERE value < 1
```

Of course, you can write a more complex script, but it's OK for the sake of simplicity. Now let's use the QAToolKit custom SQL rule and generate the scripts:

```csharp
var generator = new SqlServerTestGenerator(options =>
{
    options.AddCustomSqlRule(
        new List<string>()
        {
            "SELECT * FROM [table] WHERE timestamp = '2016-05-31'",
            "SELECT * FROM [table] WHERE value < 1"
        });
});

List<DatabaseTest> scripts = await generator.Generate();
```

And as you already know from the previous posts, now we run the scripts and collect results:

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
    "DatabaseResult": false,
    "Variable": null,
    "Script": "IF EXISTS (SELECT * FROM [table] WHERE timestamp = '2016-05-31') BEGIN Select 1 END ELSE BEGIN Select 0 END;",
    "DatabaseTestType": "CustomScript",
    "DatabaseKind": "SQLServer"
  },
  {
    "DatabaseResult": true,
    "Variable": null,
    "Script": "IF EXISTS (SELECT * FROM [table] WHERE value < 1) BEGIN Select 1 END ELSE BEGIN Select 0 END;",
    "DatabaseTestType": "CustomScript",
    "DatabaseKind": "SQLServer"
  }
]
```

Our custom SQL script then injected in the IF EXISTS - THEN -ELSE envelope as shown above. The execution of that query returns 1 or 0, true or false, depending on the result of your SQL query.

You can assert results by xUnit or NUnit, or you can process the results however you want.

Read more about the library [here](https://github.com/qatoolkit/qatoolkit-engine-database-net).