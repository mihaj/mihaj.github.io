---
layout: post
title: Powershell Core Auto-complete with PSReadline
excerpt_separator: <!--more-->
author: Miha J.
tags: powershell rsreadline
---
<!--more-->
I am a heavy user of `Powershell Core` in combination with `Windows Terminal`. Did you know you could have auto-complete in your Powershell?

Use `PSReadLine` module to achieve that.

You can install it by:

```powershell
Install-Module -Name PSReadLine -AllowPrerelease -Scope CurrentUser -Force -SkipPublisherCheck
```

Then you can change `powershell` $PROFILE to set it up every time you run the Powershell.

From `powershell` run `notepad $PROFILE` and add the content below to the end of the file:

```powershell
notepad $PROFILE
```

And add the content below to it.

```powershell
#Autocomplete
Import-Module PSReadLine
Set-PSReadLineKeyHandler -Key Tab -Function MenuComplete
Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward
Set-PSReadLineOption -ShowToolTips
Set-PSReadLineOption -PredictionSource History
```

Save and restart Powershell Core, and you'll get the auto-complete feature!