---
layout: post
title: Run ForEach loop in parallel in PowerShell Core
excerpt_separator: <!--more-->
author: Miha J.
tags: powershell
---
<!--more-->
If you are like me, writing all sorts of scripts in [PowerShell Core](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-windows?view=powershell-7.1) to automate processes, this is a post for you.

There might be times when you have a time-consuming process, and running them in sequence might be too slow. If you can put the process in the `ForEach loop`, you are lucky because there is a parallel flavor of `ForEach loop`.

PowerShell Core allows you to run the `ForEach loop` in parallel like this:

```powershell
$services = "Service1","Service1","Service3","Service4","Service5","Service6","Service7"
$tag="latest"

$services | ForEach-Object -Parallel {
    pwsh -Command "docker build -t $($_):$($using:tag) -f $($_).Dockerfile ."
} -ThrottleLimit 3
```

The script above will loop through the service list in the batch of 3 and process them in parallel. While we look at the script, we can also address two things.

#### Current variable in the loop
The `$_` is used to access the current variable in the loop, a current value from the array $services.

#### Subexpression operator $()
For example, $($_) means that the expressions inside the subexpression operator will execute first and return it as a variable.

#### Access variables with $using:
We've defined a variable `$tag` outside of the ForEach loop. In C#, for example, we do not have issues accessing that variable in the loop. But in PowerShell, you need to retrieve it by `$using`.

So `$($using:tag)` will result in value `latest`.
