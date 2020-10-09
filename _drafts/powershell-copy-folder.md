---
layout: post
title: Copy a folder to X locations
excerpt_separator: <!--more-->
author: Miha J.
tags: powershell
---

If you want to copy a folder content to X locations through iteration, you can do it like this in `powershell`:

```powershell
For ($i=0; $i -le 1000; $i++)
{
   Copy-Item -path "C:\Temp\Copy\*" -Destination "C:\inetpub\wwwroot\mysites-$($i)" -Force -Recurse
}
```

That will copy the contents of the folder `C:\Temp\Copy\*` to 1000 folders of `C:\inetpub\wwwroot\mysites-X` where X is an a loop counter. It will also do a deep copy, which means going through all the folders and overwrite the files in a destination folder.

Useful?