---
layout: post
title: Kubernetes Startup probe with .NET 6 - part 2
excerpt_separator: <!--more-->
author: Miha J.
tags: .NET, net6, c#, kubernetes, startup probe
---

In part 1, I showed you how you can create a [liveness probe](https://mihajakovac.com/2022-02-09-kubernetes-liveness-probe-with-dot-net6). In part 2, I will talk about startup probes.

The startup probe is a probe signaling a healthy service when it starts. We need to capture that information, so I've developed a solution to create an `IHosteService` implementation that will control the `IHostApplicationLifetime` state.

With the solution below, we are taking care of the probe and taking care of the Service lifetime actions, like `OnShutdown`, `OnStarted` and on `OnStopped`.

Here is the code:

```c#
using System.Threading;
using System.Threading.Tasks;
using Common.HealthChecks.AppEvents;
using Microsoft.Extensions.Hosting;

namespace Common.HealthChecks
{
   public enum ServiceState
   {
	  Shutdown,
      Stopped,
	  Started
   }
	
   public class ServiceStatus
   {
	  public ServiceState State = ServiceState.Stopped;
   }
	
   public class HostApplicationLifetimeEventsHostedService : IHostedService
   {
      private readonly IHostApplicationLifetime _hostApplicationLifetime;
      public ServiceStatus ServiceStatus { get; private set; }
      
      public HostApplicationLifetimeEventsHostedService(IHostApplicationLifetime hostApplicationLifetime)
      {
         _hostApplicationLifetime = hostApplicationLifetime;
         ServiceStatus = new ServiceStatus();
      }

      public Task StartAsync(CancellationToken cancellationToken)
      {
         _hostApplicationLifetime.ApplicationStarted.Register(OnStarted);
         _hostApplicationLifetime.ApplicationStopping.Register(OnShutdown);
         _hostApplicationLifetime.ApplicationStopped.Register(OnStopped);

         return Task.CompletedTask;
      }

      public Task StopAsync(CancellationToken cancellationToken)
         => Task.CompletedTask;

      private void OnShutdown()
      {
         for (var i = 5; i > 0; i--)
         {
            ServiceStatus.State = ServiceState.Shutdown;
            Thread.Sleep(1000);
         }
      }

      private void OnStopped()
      {
         ServiceStatus.State = ServiceState.Stopped;
      }

      private void OnStarted()
      {
         ServiceStatus.State = ServiceState.Started;
      }
   }
}
```

Next, let's add the Health Check class:

```c#
using System;
using Common.HealthChecks.HealthChecks;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Diagnostics.HealthChecks;

namespace Common.HealthChecks
{
   public static partial class HealthCheckExtensions
   {
      public static IServiceCollection AddHesStartupHealthChecks(this IServiceCollection services)
      {
         services.AddHealthChecks()
            .AddCheck<StartupHealthCheck>("Service Startup",
               HealthStatus.Unhealthy,
               new[] { Startup },
               TimeSpan.FromSeconds(3));

         return services;
      }
   }
}
```

The code registers a health check that returns healthy status if `ServiceStatus.State = ServiceState.Started;`. We also set the tag of the health check to `Startup`, which is helpful for grouping health checks together when exposing a `/startup` endpoint.

The code below exposes the `/startup` endpoint on our service and checks all health checks with the `Startup` tag.

```c#
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Diagnostics.HealthChecks;

namespace Beekn.WebApi.Extensions.HealthChecks;

public static class RegisterProbesExtensions
{
    public static void RegisterHealthCheckProbes(this IApplicationBuilder app)
    {
        app.UseHealthChecks("/startup",
         new HealthCheckOptions { Predicate = check => check.Tags.Contains(HealthCheckExtensions.Startup) });
    }
}
```

Now let's register health check and `HostApplicationLifetimeEventsHostedService` as a singleton in the startup:

```c#
public void ConfigureServices(IServiceCollection services)
{
    ...
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

Now you can run the application and go to `/startup` endpoint, which should return 200 and `Healthy` if `HostApplicationLifetimeEventsHostedService` started successfully.
