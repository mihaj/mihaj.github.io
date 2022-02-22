---
layout: post
title: Get Access Token (JWT) from Identity Server 4 with QAToolKit
excerpt_separator: <!--more-->
author: Miha J.
tags: jwt, qatoolkit, C#, access token, identity server 4, ROPC
---

<!--more-->
So far, we have covered retrieving an access token (JWT) from the Keycloak and Azure AD B2C using [QaToolKit Auth library](https://github.com/qatoolkit/qatoolkit-auth-net
). We will finish with [Identity Server 4](https://duendesoftware.com/). It's a well-known OpenID Connect and OAuth 2.0 framework in the .NET space.

As always, first, you need to add a NuGet to your C# project: `Install-Package QAToolKit.Auth`.

For Client Credential flow create `IdentityServer4Authenticator` and set the `AddClientCredentialFlowParameters` options like:

```csharp
var auth = new IdentityServer4Authenticator(options =>
{
    options.AddClientCredentialFlowParameters(
            new Uri("https://<myserver>/token"),
            "<my_client>"
            "<client_secret>");
});
            
var token = await auth.GetAccessToken();
//eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJhS3VDVVBQV1RQd19vZDdpQUpzcDRPRkoxLXM2d0I5RVgzUDAyODhXRS1FIn0.eyJleHAiOjE2MjMzMDY0NzUsImlhdCI6MTYyMzMwNDY3NSwiYX....
```

By calling `GetAccessToken()`, we get the access token. Alternatively, you can use _Resource Owner Client Credential_ flow (ROPC) by providing a username and password in the request.

```csharp
var auth = new IdentityServer4Authenticator(options =>
{
    options.AddResourceOwnerPasswordCredentialFlowParameters(
            new Uri("https://<myserver>/token"),
            "<my_client>"
            "<client_secret>",
            "user",
            "pass");
});
            
var token = await auth.GetAccessToken();
//eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJhS3VDVVBQV1RQd19vZDdpQUpzcDRPRkoxLXM2d0I5RVgzUDAyODhXRS1FIn0.eyJleHAiOjE2MjMzMDY0NzUsImlhdCI6MTYyMzMwNDY3NSwiYX....
```

Once you have an access token, you can use it in your HTTP Client requests and start testing. I suggest you check and use [QAToolKit Http Tester library](https://github.com/qatoolkit/qatoolkit-engine-httptester-net).

_Please note that ROPC flow is mainly not recommended for a production environment with real users. There are more secure flows to use. I've decided to put ROPC to the `QAToolKit Auth library` because I think this flow might be handy for testing purposes, where we control the client (testing application) and the server (application we test)._