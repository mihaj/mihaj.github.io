---
layout: post
title: Build a single executable in .NET 6.0 - update
excerpt_separator: <!--more-->
author: Miha J.
tags: net6, c#
---
A year ago, I published a blog post where I described how to publish a [.NET 6 self-contained single executable application](https://www.mihajakovac.com/build-dotnet-single-exe-net6-edition/).

I want to update the previous post for .NET 7, which came with a new dotnet publish argument, `EnableCompressionInSingleFile`. We can use it like this:

`dotnet publish My.csproj --configuration Release --framework net7.0 --self-contained True --output Publish --runtime win-x64 --verbosity Normal /property:PublishTrimmed=False /property:PublishSingleFile=True /property:IncludeNativeLibrariesForSelfExtract=True /property:DebugType=None /property:DebugSymbols=False /property:EnableCompressionInSingleFile=True`

The trimming of the self-contained assembly is time-consuming so compression might be a good alternative. If you combine both, you can get the perfect file size.

Below are all permutations of trimming and compression:

### ☒ trimming, ☒ compression - File size of 70 MB
`/property:PublishTrimmed=False /property:PublishSingleFile=True /property:IncludeNativeLibrariesForSelfExtract=True /property:DebugType=None /property:DebugSymbols=False /property:EnableCompressionInSingleFile=False`

### ☒ trimming, ☑ compression - File size of 36 MB
`/property:PublishTrimmed=False /property:PublishSingleFile=True /property:IncludeNativeLibrariesForSelfExtract=True /property:DebugType=None /property:DebugSymbols=False /property:EnableCompressionInSingleFile=True`

### ☑ Trimming, ☒ compression - File size of 25 MB
`/property:PublishTrimmed=True /property:PublishSingleFile=True /property:IncludeNativeLibrariesForSelfExtract=True /property:DebugType=None /property:DebugSymbols=False /property:EnableCompressionInSingleFile=False`

### ☑ Trimming , ☑ compression - File size of 15 MB
`/property:PublishTrimmed=True /property:PublishSingleFile=True /property:IncludeNativeLibrariesForSelfExtract=True /property:DebugType=None /property:DebugSymbols=False /property:EnableCompressionInSingleFile=True`

Please remember that compressed assembly will take longer to start, so if that is a concern in your application, test it out thoroughly.
