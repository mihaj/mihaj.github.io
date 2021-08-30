---
layout: post
title: Copy a folder to X generated folders with Powershell
excerpt_separator: <!--more-->
author: Miha J.
tags: powershell, copy
---
<!--more-->
Below is a simple Powershell script that can copy a folder to X folders through a for-loop. You can do it like this:

```powershell
For ($i=0; $i -le 1000; $i++)
{
   Copy-Item -path "C:\Temp\Source\*" -Destination "C:\Temp\Destination\Folder-$($i)" -Force -Recurse
}
```

That will copy all the contents of the folder `C:\Temp\Source` to 1000 folders of `C:\Temp\Destination\Folder-X` where X is an a loop counter.
It will also do a deep copy, going through all the folders recursively in the source folder.