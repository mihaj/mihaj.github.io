---
layout: post
title: Scroll to the bottom of the page with Selenium
excerpt_separator: <!--more-->
author: Miha J.
tags: selenium, UI tests, C#
---

<!--more-->
I will show you how to scroll to the bottom of the web page with a simple `C# do-while loop` with a `Selenium Edge browser driver` in today's nugget.

Sometimes, the web page content loads while you scroll down. The other times the web server loads the whole page at once and returns it to the browser.

In both cases, If we want to click on a link located at the bottom of the web page, we need to scroll down with Selenium driver. You want to know how? Continue reading.

1. First, we need to create a Selenium driver and point it to the URL. 
2. Then we need to create a do-while loop, check the initial scroll height, scroll down, and recheck the scroll height.
3. Finally compare before and after scroll heights and determine if the driver needs to continue scrolling or is at the bottom of the page.

```csharp
EdgeOptions options = new EdgeOptions();
options.UseInPrivateBrowsing = true;
driver = new EdgeDriver(options);

driver.Navigate().GoToUrl("https://mypage.com");

long beforeScrollHeight = 0;
do
{
    //Get current height
    beforeScrollHeight = Convert.ToInt64((long) driver.ExecuteScript("return document.body.scrollHeight"));
    
    //Scroll down
    ((IJavaScriptExecutor) driver).ExecuteScript(
        "window.scrollTo(0, document.body.scrollHeight + 500)");

    //Sleep
    Thread.Sleep(1000);
    
    //Get scroll height after the scroll
    afterScrollHeight = Convert.ToInt64((long) driver.ExecuteScript("return document.body.scrollHeight"));
} while (afterScrollHeight != beforeScrollHeight);
```

Ta-da!