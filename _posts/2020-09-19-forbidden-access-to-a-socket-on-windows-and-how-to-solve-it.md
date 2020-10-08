---
layout: post
title: Forbidden access to a socket on Windows and how to solve it
excerpt_separator: <!--more-->
author: Miha J.
tags: windows hyper-v
---

Lately I was getting unfamiliar errors when running a .NET applications or spawning docker container on Windows. I was really annoyed I could not run the application on port 5001 from Visual Studio, since it was working fine just a day before.

The core of the message I got was: `An attempt was made to access a socket in a way forbidden by its access permissions`.

"Which application is using this port?" I did a "good old" computer restart ... and ... nothing.
<!--more-->
I run a `netstat -ano` which lists all computer connections and ports, and the port was not there. "Huh?"

After a quick Google search I found a command, that can show me "reserved" or excluded ports on my Windows instance.

The command is `netsh interface ipv4 show excludedportrange protocol=tcp`:

```
Protocol tcp Port Exclusion Ranges

Start Port    End Port
----------    --------
        80          80
        81          81
        88          88
       443         443
      2869        2869
      4041        5040     
      5041        5140
      5141        5240
      5241        5340
      5357        5357
      8000        8000
     50000       50059     *

* - Administered port exclusions.
```

"Aha, there you are! How come you are suddenly reserved?" Line 10 shows a reserved port range which includes my bellowed port! I've used this port for ages and never had any issues. It looks like some update or upgrade to the system or application did that. After a few more Google searches and "strength training with my coffee mug" I found a promising post on [Github](https://github.com/docker/for-win/issues/3171). As you'll soon find out the culprit was the *Hyper-V* which for some reason reserved the ports for itself. I suspect a Docker for Windows update was the reason in this case.

Anyway, below is a compiled solution to my problem.

### Solution

Open the CMD or Powershell as *administrator* and run the commands below.

##### 1. Disable Hyper-V

First disable the Hyper-V and restart a computer:

`dism.exe /Online /Disable-Feature:Microsoft-Hyper-V`

##### 2. Exclude the ports from the list of available ports that can be reserved (by Hyper-V in my case)

Then if you run the `netsh interface ipv4 show excludedportrange protocol=tcp` again you'll see a different results from before. Hyper-V was disabled and the ports were released. Awesome! Let's exclude the ports. The command below will exclude 10 consecutive ports starting from 5000.

`netsh int ipv4 add excludedportrange protocol=tcp startport=5000 numberofports=10`

What if I get the error `The process cannot access the file because it is being used by another process`?

Adjust the command to exclude a smaller range of ports. One of your applications is using one or more of those 10 ports that you would like to exclude.

##### 3. Enable Hyper-V

Now let's enable the Hyper-V and restart!

`dism.exe /Online /Enable-Feature:Microsoft-Hyper-V /All`

##### 4. Check if port is excluded

Run the `netsh interface ipv4 show excludedportrange protocol=tcp` again:

```
Protocol tcp Port Exclusion Ranges

Start Port    End Port
----------    --------
	  ....   	  ....    
      5000        5010	   *
	  ....   	  ....  
     50000       50059     *

* - Administered port exclusions.
```

Yes! I can run my application on port 5001 again! "WOOOHOOOO" :)!