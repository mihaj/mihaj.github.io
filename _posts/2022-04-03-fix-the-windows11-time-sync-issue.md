---
layout: post
title: Fix the Windows 11 time sync issue
excerpt_separator: <!--more-->
author: Miha J.
tags: windows11, time sync
---

Windows time was lately out of order. The time was wrong, and I wondered why.

I tried to sync it within the Windows `Date&Time` dashboard, but it did not work.
- I restarted the Windows Time service, and it did not work.
- If I set the clock manually, it was off by 1 hour.
- I even replaced the battery on my motherboard, as some sources suggested.

Nothing worked!

Then I decided to choose a different NTP server. I went to a [pool.ntp.org](https://www.pool.ntp.org/) website and got my local NTP server: https://www.pool.ntp.org/zone/si.

In the PowerShell, I typed:

```powershell
w32tm /config /manualpeerlist:si.pool.ntp.org /update
w32tm /resync
```

And my time was correct again! Phew!

