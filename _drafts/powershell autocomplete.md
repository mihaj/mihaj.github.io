---
layout: post
title: Powershell Core Autocomplete with PSReadline
excerpt_separator: <!--more-->
author: Miha J.
tags: powershell rsreadline
---

I am heavy user of `Powershell Core` in combination with `Windows Terminal`. Did you know you could have auto-complete in your Powershell?

That is achieved by a module `PSReadLine`.

You can install it by:

```

```

Then you can change `powershell` $PROFILE to set it up every time you run the Powershell.

From `powershell` run `notepad $PROFILE` and add the content below to the end of the file:

```powershell
#Autocomplete
Import-Module PSReadLine
Set-PSReadLineKeyHandler -Key Tab -Function MenuComplete
Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward
Set-PSReadLineOption -ShowToolTips
Set-PSReadLineOption -PredictionSource History
```

Save and restart and you'll get the auto-complete feature!