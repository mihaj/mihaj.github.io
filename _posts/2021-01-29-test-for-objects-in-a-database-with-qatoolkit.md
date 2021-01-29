---
layout: post
title: Test for objects in a database with QAToolKit
excerpt_separator: <!--more-->
author: Miha J.
tags: c# tool qatoolkit database
---

I find it necessary for us to test our databases and do that in an automated fashion. Sometimes, we want to check that we migrated a new table successfully or removed that obsolete stored procedure. Other times we want to check if the records exist in the database.

With the [QAToolKit Database library](https://github.com/qatoolkit/qatoolkit-engine-database-net), we can quickly implement these kinds of tests. Let's check how we can test if a table, view, and stored procedure exist in the database.

First, we define rules and generate scripts:
```csharp
var generator = new SqlServerTestGenerator(options =>
{
    options.AddDatabaseObjectExitsRule(new string[] { "Users" }, DatabaseObjectType.Table);
    options.AddDatabaseObjectExitsRule(new string[] { "SearchUsers" }, DatabaseObjectType.View);
    options.AddDatabaseObjectExitsRule(new string[] { "AddNewUser" }, DatabaseObjectType.StoredProcedure);
}
```

Then we execute those scripts against database and gather results:
```csharp
var runner = new SqlServerTestRunner(scripts, options =>
{
    options.AddSQLServerConnection("server=localhost;user=user;password=mypassword;Initial Catalog=myDatabase");
});

List<DatabaseTestResult> results = await runner.Run();
```

One note. The above example is working on the **Microsoft SQLServer** database, but you can also use MySQL or Postgresql servers. Read more [here](https://github.com/qatoolkit/qatoolkit-engine-database-net).