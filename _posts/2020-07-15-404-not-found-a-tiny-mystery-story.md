---
layout: post
title: 404 Not Found – A tiny mystery story
excerpt_separator: <!--more-->
author: Miha J.
---

Occasionally you get an error or found an issue in your code, which you can not comprehend. It can be difficult to solve, or it is something really silly. In any case that is when we put our detective hats on and start the investigation.

Lately, we were upgrading our customer’s Web API to a v2 and everything went well. We changed the URI paths, implemented new logic, and move to the next endpoint. But a colleague pointed out an endpoint which always returned 404 – Not found error. No problem, that is easy!
<!--more-->
“Check the routing path, there must be a typo in there!” - Nope.

“Check the Postman settings!” - Nope.

Let's put our detective hats on and be really smart! And then ... we tried things that did not make any sense.

“Check the… check the authentication!” - Why? The endpoint is located in the same controller with the same Authentication settings works fine! And it should return 401!

“Check the start-up pipeline! There must be some mix-up of middle-ware!” – It is fine, we did not change anything from the last working version!

“Change the route to something else and check if that works!” – Nope, it still does not work.

“Move the endpoint to a different controller with fewer dependencies!” – What the….what are we doing...

“OK, let’s sleep it over…”

The next day, we were fresh and filled with morning enthusiasm and continued with the troubleshooting. Luckily for us, we started to use much better approaches! A colleague suggested to use Fiddler to diagnose the request and viola, he got back a funny surprise!

```
GET https://localhost:5003/api/call%E2%80%8B?api-version=2.0 HTTP/1.1
```

“What is that appendix at the end of /call?” “Emm, I copy and pasted the route from the endpoint and called it with Postman.” Some characters sneaked in out URI path?! Let us check the path. We opened our controller in Visual Studio and the definition “looks fine".

```
[HttpGet(“call”)]
```

"Ahhhh, let us change the encoding of the file from UTF-8 to ANSI and see what we get."

```
[HttpGet(“callâ€‹")]
```

Woohooo! We got you! High five! And that’s how we solved the 404 mystery and rode off into the day!