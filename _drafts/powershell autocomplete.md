---
layout: post
title: Powershell Core Autocomplete with PSReadline
excerpt_separator: <!--more-->
author: Miha J.
tags: powershell rsreadline
---

Change `powershell` $PROFILE, by adding script below. When Powershell core is started the profile will be applied.
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