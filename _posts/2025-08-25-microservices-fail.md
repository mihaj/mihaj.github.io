---
layout: post
title: Microservices fail
excerpt_separator: <!--more-->
author: Miha J.
tags: microservices, modular monoliths
---

𝐌𝐨𝐬𝐭 𝐦𝐢𝐜𝐫𝐨𝐬𝐞𝐫𝐯𝐢𝐜𝐞 𝐩𝐫𝐨𝐣𝐞𝐜𝐭𝐬 𝐝𝐨𝐧’𝐭 𝐟𝐚𝐢𝐥 𝐛𝐞𝐜𝐚𝐮𝐬𝐞 𝐨𝐟 𝐭𝐡𝐞 𝐭𝐞𝐜𝐡.

Teams often overengineer with microservices too early - before the domain is clear. Add to that unclear module dependencies or inefficient communication between services, and collaboration quickly breaks down.

The result? Slower delivery timelines and growing technical debt.

I implement this differently by 𝐢𝐧𝐭𝐞𝐫𝐦𝐨𝐝𝐮𝐥𝐞 𝐜𝐨𝐦𝐦𝐮𝐧𝐢𝐜𝐚𝐭𝐢𝐨𝐧 for .NET modules (𝘯𝘶𝘨𝘦𝘵 𝘱𝘢𝘤𝘬𝘢𝘨𝘦s) that solves the problem:

- Modules in a 𝐦𝐨𝐝𝐮𝐥𝐚𝐫 𝐦𝐨𝐧𝐨𝐥𝐢𝐭𝐡 𝐭𝐚𝐥𝐤 𝐢𝐧-𝐦𝐞𝐦𝐨𝐫𝐲.
- The same modules, when run as 𝐦𝐢𝐜𝐫𝐨𝐬𝐞𝐫𝐯𝐢𝐜𝐞𝐬, 𝐭𝐚𝐥𝐤 𝐯𝐢𝐚 𝐠𝐑𝐏𝐂.
- No code changes are needed when you switch - start as a monolith, break out into services only when it makes sense.

This way you move fast early, then scale your architecture only when the domain is clear - without paying the rewrite tax.
