---
layout: post
title: Write a simple service status checker
excerpt_separator: <!--more-->
author: Miha J.
tags: c# monitoring
---

I like using tools. I also like writing me some nice tools. The one I will present today is about checking the heatlh of HTTP API endpoints.

It's a simple tool that checks wheter a list of API URLs return 200.

So in a nutshell I have a list of API services. The target is `/hc` endpoint and is a readyness probe. This means that the code behind the endpoint is already checking a lot for us. From database connections, 3rd party services, etc. And if the `/hc` returns 200 - OK, that means more than just `/hc` is online.

Ok, here is the core of the tool:

```csharp
var hcs = new string[] {
    $"https://users{environment}.api.com/hc",
    $"https://orders{environment}.api.com/hc",
    $"https://sites{environment}.api.com/hc",
    $"https://files{environment}.api.com/hc"
    };

foreach (var service in hcs)
{
    using (var client = new HttpClient())
    {
        client.BaseAddress = new Uri(service);
        var response = client.GetAsync("").GetAwaiter().GetResult();
        if (response.IsSuccessStatusCode)
        {
            Console.WriteLine($"[OK] - {Convert.ToInt16(response.StatusCode)} - {service}");
        }
        else
        {
            Console.WriteLine($"[FAILED] - {Convert.ToInt16(response.StatusCode)} - {service}");
        }
    }
}
```

A hawk eye might spoted I am using `{environment}` variable in the URL list. That's because I can easily test multiple environments! Check this Github repostory for complete solution.

At the end I can run something like: `checkapi dev status` or `checkapi prod status` and get back a quick service status:

```
> checkapi dev status
[OK] - 200 - https://users-dev.api.com/hc
[OK] - 200 - https://orders-dev.api.com/hc
[OK] - 200 - https://sites-dev.api.com/hc
[OK] - 200 - https://files-dev.api.com/hc
```

You can go many steps further and get back more data and make the tool more comprehensive than jsut checking 200 OK.