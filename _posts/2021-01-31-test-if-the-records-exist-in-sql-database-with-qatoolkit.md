---
layout: post
title: Test if the records exist in SQL database with QAToolKit
excerpt_separator: <!--more-->
author: Miha J.
tags: c# tool qatoolkit database sql
---

In the previous post, we looked into how we can test for tables, views, and stored procedures in the SQL database. Now, let's continue with [QAToolKit Database library](https://github.com/qatoolkit/qatoolkit-engine-database-net) and test for specific records in a database.

First, we define rules and generate scripts. The below rule will generate an SQL script that will check if the record with the name _myname_ exists in the table _mytable_:
```csharp
var generator = new SqlServerTestGenerator(options =>
{
    options.AddDatabaseRecordExitsRule(new List<DatabaseRecordExistRule>()
    {
        new DatabaseRecordExistRule()
        {
            TableName = "mytable",
            ColumnName = "name",
            Operator = "=",
            Value = "myname"
        }
    });
}
```

To execute the script we create the _runner_ and run it:
```csharp
var runner = new SqlServerTestRunner(scripts, options =>
{
    options.AddSQLServerConnection("server=localhost;user=user;password=mypassword;Initial Catalog=myDatabase");
});

List<DatabaseTestResult> results = await runner.Run();
```

This is the `results` object we get:

```json
[
  {
    "DatabaseResult": false,
    "Variable": "mytable",
    "Script": "IF EXISTS (SELECT * FROM [mytable] WHERE [name] = 'myname') BEGIN Select 1 END ELSE BEGIN Select 0 END;",
    "DatabaseTestType": "RecordExist",
    "DatabaseKind": "SQLServer"
  }
]
```

`DatabaseResult` is `false`, which means the script returned 0 - we did not find the record in a table _mytable_. Additionally the runner returns `DatabaseTestType` and `DatabaseKind` parameters.

Read more about the library [here](https://github.com/qatoolkit/qatoolkit-engine-database-net).