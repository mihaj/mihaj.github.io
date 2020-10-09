---
layout: post
title: Need a generated key?
excerpt_separator: <!--more-->
author: Miha J.
tags: c#
---

Need a generated key for your password or api key?

Open up a `C# interactive console` in Visual Studio and paste this lines:

```csharp
using System.Security.Cryptography;
var key = new byte[32];
using (var generator = RandomNumberGenerator.Create())
 generator.GetBytes(key);
return Convert.ToBase64String(key);
```

The returned value is a string like this:

`KT7fdJ9pilnOMuubWc3oUHNjhXBylTRqGGyRXLnR30s=`
