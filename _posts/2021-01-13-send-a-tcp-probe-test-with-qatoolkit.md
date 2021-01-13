---
layout: post
title: Send a TCP probe test with QAToolKit
excerpt_separator: <!--more-->
author: Miha J.
tags: c# tool qatoolkit tcp
---

Lately, I've been working on tests that needed to connect to the service through the TCP protocol. The service was accepting a command and returning a response with results.

<!--more-->

To achieve that, you can use the [QAToolKit Probes library](https://github.com/qatoolkit/qatoolkit-engine-probes-net) to create a **TCP Probe**, which is an excellent addition to the [HTTP](https://www.mihajakovac.com/liveness-check-tool-for-your-http-services/) and [ICMP](https://www.mihajakovac.com/use-qatoolkit-to-create-ping-tests/) tests we looked at earlier.

The code below shows how easy it is to create a new TCP probe and send a message. You can also send a list of statements depending on your needs. The probe processes them in sequence and returns a response message.

```csharp
//Copy and paste code
using QAToolKit.Engine.Probes.Probes;
using System;
using System.Threading.Tasks;

namespace ConsoleApp3
{
    class Program
    {
        private static async Task Main(string[] args)
        {           
                var tcpProbe = new TcpProbe("tcpbin.com", 4242, new[] { "HELLO\n" });
                var result = await tcpProbe.Execute();
                Console.WriteLine($"tcpbin.com:4242 responded with {result.ResponseData}");
        }
    }
}
```

And here is the output:

```
tcpbin.com:4242 responded with HELLO
```

As always, you can also use QAToolKit to write xUnit or NUnit tests. Simple & useful. :)