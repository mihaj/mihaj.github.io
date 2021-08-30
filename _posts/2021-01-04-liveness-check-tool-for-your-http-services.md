---
layout: post
title: Liveness check tool for your HTTP services
excerpt_separator: <!--more-->
author: Miha J.
tags: c# tool qatoolkit http
---
<!--more-->
I like using tools. And as you may already know, I also enjoy writing tools. The one I will present today is about checking the **health of HTTP API endpoints**. It's a simple tool that returnes the HTTP endpoint status code.

OK, here is the core code of the tool:

```csharp
//Copy and paste code
using QAToolKit.Engine.Probes.Probes;
using System;
using System.Net.Http;
using System.Threading.Tasks;

namespace MyApp
{
    class Program
    {
        private static async Task Main(string[] args)
        {
            var hcs = new string[] {
                $"https://users-dev.api.com/hc",
                $"https://orders-dev.api.com/hc",
                $"https://sites-dev.api.com/hc",
                $"https://files-dev.api.com/hc"
            };

            foreach (var service in hcs)
            {
                var httpProbe = new HttpProbe(
                    new Uri(service),
                    HttpMethod.Get);
                var result = await httpProbe.Execute();
                Console.WriteLine($"[{result.StatusCode}] - {service}");
            }
        }
    }
}

```

15 lines of code! We are using [QAToolKit's Network Probes](https://github.com/qatoolkit/qatoolkit-engine-probes-net) [Nuget Package](https://www.nuget.org/packages/QAToolKit.Engine.Probes/), to create **HTTP probes**. Here is the output:

```
[OK] - https://users-dev.api.com/hc
[OK] - https://orders-dev.api.com/hc
[OK] - https://sites-dev.api.com/hc
[OK] - https://files-dev.api.com/hc
```

Simple and useful. :)