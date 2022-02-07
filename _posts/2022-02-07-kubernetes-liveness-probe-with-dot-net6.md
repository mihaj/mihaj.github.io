---
layout: post
title: Kubernetes liveness probe with .NET 6 - part 1
excerpt_separator: <!--more-->
author: Miha J.
tags: .NET, net6, c#, kubernetes, liveness probe
---

The Kubernetes is a container orchestrator that sweeps through the cloud-native world application development. For orchestrator to manage our applications, it needs to know its state, like:

- Did my application start?
- Is my application running?
- Is my application ready to be used?

The Kubernetes (or other systems) can ping all of those endpoints every X seconds and determine what is the state of our application. 

In the next 3 posts, you will read about my interpretations of `/startup`, `/liveness` and `/readiness` [probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) and how to implement them with C# and .NET 6.

First, we will start with the `Liveness` probe.

The liveness probe detects if the application is running and responding and returns the correct response in the case of HTTP API. Great libraries are available in .NET, which give us the health checking infrastructure.

To start implementing the liveness probe in .NET 6, you don't need to reference any additional library. The `Microsoft.Extensions.Diagnostics.HealthChecks` should be included in the base framework.

Here is the code for the liveness health check:

```c#
using System;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Diagnostics.HealthChecks;

namespace Beekn.WebApi.Extensions.HealthChecks;

public static partial class HealthCheckExtensions
{
    public static IServiceCollection AddSelfCheck(this IServiceCollection services)
    {
        services.AddHealthChecks()
            .AddCheck("Self test",
                () => HealthCheckResult.Healthy(),
                new[] { Liveness },
                TimeSpan.FromSeconds(3));
        return services;
    }
}
```

The code registers a health check that returns healthy status if it executes. We also set the tag of the health check to `Liveness`, which is helpful for grouping health checks together when exposing a `/liveness` endpoint.

The code below exposes the `/liveness` endpoint on our service and checks all health checks with a tag `Liveness`. Tags are much more helpful in a `/readiness` probe, which I will cover in part 3 of this tutorial.

```c#
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Diagnostics.HealthChecks;

namespace Beekn.WebApi.Extensions.HealthChecks;

public static class RegisterProbesExtensions
{
    public static void RegisterHealthCheckProbes(this IApplicationBuilder app)
    {
        app.UseHealthChecks("/liveness",
            new HealthCheckOptions { Predicate = check => check.Tags.Contains(HealthCheckExtensions.Liveness) });
    }
}
```

Now let's call both functions in the startup:

```c#
public void ConfigureServices(IServiceCollection services)
{
    ...
    services.AddSelfCheck();
    ...
}

public void Configure(IApplicationBuilder app)
{
    ...      
    app.RegisterHealthCheckProbes();
    ...
}
```

And that's it. Now you can run the application and go to `/liveness` endpoint, which should return 200 and `Healthy`.