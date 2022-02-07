---
layout: post
title: Kubernetes Readiness probe with .NET 6 - part 3
excerpt_separator: <!--more-->
author: Miha J.
tags: .NET, net6, c#, kubernetes, readiness probe
---

In part 3, we will look into `readiness probe` endpoint. The readiness probe endpoint is a must for any system, that relies on other infrastructure, like databases, messaging systems, other micro services, cloud services like file storages, Redis, etc.

That is important, because if some of those dependant services are not running our services do not work properly. *We can easily signal, that our service is operational or not*.

For sake of simplicity, I will focus on database health check and include it in the `readiness` probe endpoint. Let's create it:

```c#
using System;
using Common.HealthChecks.HealthChecks;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Diagnostics.HealthChecks;

namespace Project.HealthChecks
{
	public static partial class HealthCheckExtensions
	{
		public static IServiceCollection AddHesSqlServerHealthChecks(this IServiceCollection services,
			string connectionString)
		{
			services.AddHealthChecks()
				.AddSqlServer(connectionString,
					"SELECT 1;",
					$"SQL Server",
					HealthStatus.Unhealthy,
					new string[] { "sqlserver", Readiness },
					TimeSpan.FromSeconds(3));

			return services;
		}
	}
}
```
A health check will run the `SELECT 1;` SQL query on the database specified in the database connection string (`connectionString`).

There is already a library for writing health checks for SQL Server database (`AspNetCore.HealthChecks.SqlServer`). Check more libraries [here](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks).

Now we register the `readiness` probe endpoint.

```c#
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Diagnostics.HealthChecks;

namespace Project.HealthChecks
{
	public static class RegisterProbesExtensions
	{
		public static void RegisterHealthCheckProbes(this IApplicationBuilder app)
		{
			app.UseHealthChecks("/readiness",
				new HealthCheckOptions { Predicate = check => check.Tags.Contains(HealthCheckExtensions.Readiness) });
		}
	}
}
```

And as always, register the health check in startup.

```c#
public void ConfigureServices(IServiceCollection services)
{
    ...
    services.AddHesSqlServerHealthChecks(connectionString);
    ...
}

public void Configure(IApplicationBuilder app)
{
    ...      
    app.RegisterHealthCheckProbes();
    ...
}
```

Now you can run the application and go to `/readiness` endpoint, which should return 200 and `Healthy` SQL database is accessible successfully.