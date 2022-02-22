---
layout: post
title: Kubernetes Readiness probe with .NET 6 - part 3
excerpt_separator: <!--more-->
author: Miha J.
tags: .NET, net6, c#, kubernetes, readiness probe
---

In part 1, I showed you how to create a [liveness probe](https://www.mihajakovac.com/kubernetes-liveness-probe-with-dot-net6/). In part 2, [startup probes](https://www.mihajakovac.com/kubernetes-startup-probe-with-dot-net6/).

In part 3, we will look into `readiness probe` endpoint. The readiness probe endpoint is a must for any system, that relies on other infrastructure, like databases, messaging systems, other micro services, cloud services like file storages, Redis, etc.

That is important, because if some of those dependant services are not running our services do not work properly. *We can easily signal, that our service is operational or not by implementing readiness probe endpoint*.

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
A health check will run the `SELECT 1;` SQL query on the database specified in the database connection string (`connectionString`). If the health check can not execute the query the health check will fail, meaning Kubernetes readiness probe will fail and our service will be marked as `Unhealthy`.

There is already a base library for writing health checks for SQL Server database (`AspNetCore.HealthChecks.SqlServer`). Check more libraries [here](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks).

Now let's register the `readiness` probe endpoint.

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

Now we can combine all three health check endpoints and register them like this:
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

            app.UseHealthChecks("/startup",
                new HealthCheckOptions { Predicate = check => check.Tags.Contains(HealthCheckExtensions.Startup) });
                
            app.UseHealthChecks("/liveness",
                new HealthCheckOptions { Predicate = check => check.Tags.Contains(HealthCheckExtensions.Liveness) });
        }
	}
}
```

And

```c#
public void ConfigureServices(IServiceCollection services)
{
    ...
    services.AddHesSqlServerHealthChecks(connectionString);
    services.AddSelfCheck();
    services.AddHesStartupHealthChecks();
    services.AddSingleton<HostApplicationLifetimeEventsHostedService>();
    services.AddHostedService(p => p.GetRequiredService<HostApplicationLifetimeEventsHostedService>());
    ...
}

public void Configure(IApplicationBuilder app)
{
    ...      
    app.RegisterHealthCheckProbes();
    ...
}
```