---
layout: post
title: Azure Web App for Linux Containers timeouts on start-up
excerpt_separator: <!--more-->
author: Miha J.
tags: azure, linux web apps, c#, docker
---

Lately, I have been dealing with deployment to [Azure Web Apps for Linux containers](https://azure.microsoft.com/en-us/services/app-service/containers/#overview). It's a simple-to-use service for running Linux containers.

But with easy-to-use Azure Web App service came issues, which I first credited to Azure service. And I was wrong.

I've created a .NET 5 Web API project and run it on my development machine. It worked fine until I wanted to deploy the application to Azure Web App for Linux containers. When doing that, I spotted an issue every time the Azure Web App ran the container. The application was listening on port 5000:
```
Now listening on: http://localhost:5000
```
Why 5000? Where did I set that? I went back to my development machine and checked there. It was the same thing. The only difference was that it listened in HTTP and HTTPS ports 5000 and 5001, respectively. I could not change this behavior by changing the ports like I usually did in a profile of `launchSettings.json` or by providing `ASPNETCORE_URLS` and `NETCORE_URLS` environment parameters. For example:
```
ASPNETCORE_URLS "http://+:8080"
NETCORE_URLS "http://+:8080"
```

It did not help. The app was still listening on the 5000 and 5001 ports. Is that .NET 5 specific? Am I doing something wrong? I am not sure and would sure like to know :).

After some searching online, I found that I could also specify URLs differently as `urls` parameter in an `appsettings.json` or as an environment variable.
appsettings.json
```json
{
  "urls": "http://+:80;https://+:5001",
...
```

And that did the trick! My app was not listening on ports 80 and 5001 on my development machine.

Before I went back to the Azure portal, I exposed a port 80 in the Dockerfile. I do not need an HTTPS port there.

```dockerfile
EXPOSE 80
```

And on the Azure portal, I set the `urls` parameter in the `Configuration > Application Settings > New application settings` and set it like this:

```json
 {
    "name": "urls",
    "value": "http://+:80",
    "slotSetting": false
  },
```

One setting that I find helpful is the `WEBSITES_CONTAINER_START_TIME_LIMIT`, which specifies the time limit for Web App to wait for the container to start up. A Web App will try to ping your container at port 80 and determine if it's up and ready to be served. After that time limit, the Web App timeouts and retries. The default value is 230 seconds, but I set it to 60 seconds. It's enough for my application. You can set it up at `Configuration > Application Settings > New application settings`.

An excellent resource on [GitHub](https://github.com/MicrosoftDocs/azure-docs/issues/34451) might help you if you have similar issues. It looks like this topic has been dragging along for a few years now!