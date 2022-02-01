---
layout: post
title: Build a single executable in .NET 6.0 - update
excerpt_separator: <!--more-->
author: Miha J.
tags: net6, c#
---
A few months ago, I published a blog post where I described how to publish a [.NET 5 self-contained single executable application](https://www.mihajakovac.com/build-dotnet-single-exe/).

I want to update the previous post for .NET 6, which came with a new dotnet publish argument, `EnableCompressionInSingleFile`. We can use it like this:

`dotnet publish My.csproj --configuration Release --framework net6.0 --self-contained True --output Publish --runtime win-x64 --verbosity Normal /property:PublishTrimmed=False /property:PublishSingleFile=True /property:IncludeNativeLibrari
esForSelfExtract=True /property:DebugType=None /property:DebugSymbols=False /property:EnableCompressionInSingleFile=True`

The trimming of the self-contained assembly is time-consuming so compression might be a good alternative. If you combine both, you can get the perfect file size.

### ☒ trimming, ☒ compression
A command with `/property:PublishTrimmed=False /property:PublishSingleFile=True /property:IncludeNativeLibrari
esForSelfExtract=True /property:DebugType=None /property:DebugSymbols=False /property:EnableCompressionInSingleFile=True` produced file size of **70 MB˙**.

### ☒ trimming, ☑ compression
A command with `/property:PublishTrimmed=False /property:PublishSingleFile=True /property:IncludeNativeLibrari
esForSelfExtract=True /property:DebugType=None /property:DebugSymbols=False /property:EnableCompressionInSingleFile=True` produced file size of **36 MB**.

### ☑ Trimming, ☒ compression
A command with `/property:PublishTrimmed=False /property:PublishSingleFile=True /property:IncludeNativeLibrari
esForSelfExtract=True /property:DebugType=None /property:DebugSymbols=False /property:EnableCompressionInSingleFile=True` produced file size of **25 MB**.

### ☑ Trimming , ☑ compression
`/property:PublishTrimmed=True /property:PublishSingleFile=True /property:IncludeNativeLibrari
esForSelfExtract=True /property:DebugType=None /property:DebugSymbols=False /property:EnableCompressionInSingleFile=True` produced file size of **15 MB**.

Please remember that compressed assembly will take longer to start, so if that is a concern in your application, test it out thoroughly.
