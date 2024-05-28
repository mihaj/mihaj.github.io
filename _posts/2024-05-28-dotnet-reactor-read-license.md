---
layout: post
title: How to read .NET Reactor license programmatically
excerpt_separator: <!--more-->
author: Miha J.
tags: dotnet, .net reactor
---

[.NET Reactor](https://www.eziriz.com/dotnet_reactor.htm) is a code protection and licensing system for .NET applications. It has SDK, that supports maintaining licenses within your devops toolkits.

I will not cover the license generation part in this post, but you can check the [documentation](https://www.eziriz.com/help/index.html) for more details.

Normally .NET Reactor produces `*.license` files that can be put in the same folder as your protected application. If you want to examine the license, you can use the GUI, but I will focus on C# SDK approach.

To read a license file, you need to:

1. Add a NuGet package `Eziriz.Reactor.LicenseGenerator` to your project.
2. Have `ProjectFile` with `MasterKey` ready. If your `MasterKey` is not part of the project file, you need to load it separately. MasterKey is needed to decrypt the license file.
3. Add this short piece of code:

```csharp
public void ViewLicenseInformation(string projectFile, string licenseFile)
{
    //load project file with LicenseGenerator
    var licenseReader = new LicenseGenerator(projectFile);
    
    //load license file you want to inspect
    licenseReader.LoadLicenseFromFile(licenseFile);
    
    //Read license property, here is a simple example
    Console.WriteLine($"ExpiryDate: {licenseReader.ExpirationDate}");
    Console.WriteLine("Reading Additional Values:");
    
    foreach (DictionaryEntry value in licenseReader.KeyValueTable)
    {
        Console.WriteLine($"{value.Key}: {value.Value}");
    }
}
```

This will print out ExpiryDate and two additional custom values:

```powershell
ExpiryDate: 31/12/2024 00:00:00
Company: My Company
Property1: 100000000
```

I hope this helps :).