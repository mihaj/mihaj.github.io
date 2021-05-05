---
layout: post
title: Build a "true" single executable in .NET 5.0 for Windows, Linux and MacOs
excerpt_separator: <!--more-->
author: Miha J.
tags: net5
---

Have you tried to publish a single executable file in .NET 5.0, and the result was not a single file?

For example, running this command:

`dotnet publish My.csproj --configuration Release --framework net5.0 --output publish --self-contained True --runtime win-x64 --verbosity Normal /property:PublishTrimmed=True /property:PublishSingleFile=True`

Did your output look something like this? Read further to generate a true single EXE, and it's easy.

```powershell
-a---          16/03/2021    23:02         747920 clrcompression.dll
-a---          16/03/2021    23:02        1322384 clrjit.dll
-a---          16/03/2021    23:03        5153168 coreclr.dll
-a---          05/05/2021    16:11       20971736 My.exe
-a---          05/05/2021    16:11          11088 My.pdb
-a---          16/03/2021    23:02        1056632 mscordaccore.dll
```

What are those files?

`clrcompression.dll`: native compression algorithms.
`clrjit.dll`: JIT compiler.
`coreclr.dll`: runtime.
`mscordaccore.dll`: to enable Watson dumps.
`My.pdb`: debugging symbols file to allow debugging the code.

Let's produce a single executable with the above files bundled in. We will skip the production of the PDB file. We added 3 additional parameters in the default command `/property:IncludeNativeLibrariesForSelfExtract=True /property:DebugType=None /property:DebugSymbols=False`. The end command looks like this:

`dotnet publish My.csproj --configuration Release --framework net5.0 --output publish --self-contained True --runtime win-x64 --verbosity Normal /property:PublishTrimmed=True /property:PublishSingleFile=True /property:IncludeNativeLibrariesForSelfExtract=True /property:DebugType=None /property:DebugSymbols=False`

And we only have one executable now:

```powershell
-a---          05/05/2021    16:23       29251959 My.exe
```

To create a Linux or macOS version of the executable, replace the `--runtime` parameter. For example `--runtime linux-x64` and `--runtime osx.11.0-x64` respectively.

A keen eye also notices I was using trimming (`/property:PublishTrimmed=True`) to remove unused code from the final single executable. If you have trouble publishing your application, try to remove it from the command.

References:
[.NET Single file design documentation](https://github.com/dotnet/designs/blob/main/accepted/2020/single-file/design.md)