---
layout: post
title: Use QAToolKit to create ping tests
excerpt_separator: <!--more-->
author: Miha J.
tags: c# tool qatoolkit ping
---
<!--more-->
In the [previous post](https://www.mihajakovac.com/liveness-check-tool-for-your-http-services/), we used the [QAToolKit's Network Probes](https://github.com/qatoolkit/qatoolkit-engine-probes-net) library to test the liveness of our HTTP services.

Now we will use the same library but a different probe - **Ping Probe**. Ping sends an ICMP echo request packet to the target host and waiting for an ICMP echo reply. This way, we can quickly test if a target host is up and can respond to Ping.

The QAToolKit library abstracts some of the code away and makes our code more compact. Here is a simple demonstrational program that will ping three hosts and addresses from the list.

```csharp
//Copy and paste code
using QAToolKit.Engine.Probes.Probes;
using System;
using System.Threading.Tasks;

namespace ConsoleApp2
{
    class Program
    {
        private static async Task Main(string[] args)
        {
            var hcs = new string[] {
                "google.com",
                "mihajakovac.com",
                "8.8.8.8"
            };

            foreach (var host in hcs)
            {
                var httpProbe = new PingProbe(host);
                var result = await httpProbe.Execute();
                Console.WriteLine($"{result.Success} = {result.RoundTripTime}ms - {host}");
            }
        }
    }
}
```

And here is the output:

```
Success = 17ms - google.com
Success = 193ms - mihajakovac.com
Success = 16ms - 8.8.8.8
```

You could also use that in your xUnit or NUnit tests. Simple & useful. :)