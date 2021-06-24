---
layout: post
title: Get Access Token (JWT) from Azure AD B2C with QAToolKit
excerpt_separator: <!--more-->
author: Miha J.
tags: jwt, qatoolkit, C#, access token, azure ad b2c, ROPC
---
In previous posts, we covered Keycloak and how to retrieve the JWT token from it with [QAToolKit Auth library](https://github.com/qatoolkit/qatoolkit-auth-net).

In this post, we will look into Azure AD B2C identity provider and integration with QAToolkit. As with Keycloak, we can use _Client Credential_ and _Resource Owner Password Credential_ flows to get the token. Currently, the token exchange is not supported with Azure AD B2C. Now, let's go to the code.

First, you need to add a NuGet to your C# project: `Install-Package QAToolKit.Auth`.

For Client Credential flow create `AzureB2CAuthenticator` and set the `AddClientCredentialFlowParameters` options:

```csharp
var auth = new AzureB2CAuthenticator(options =>
{
    options.AddClientCredentialFlowParameters(
            new Uri("https://login.microsoftonline.com/<tenantID>/oauth2/v2.0/token"),
            "<clientId>"
            "<clientSecret>"
            new string[] { "https://graph.windows.net/.default" });
});
            
var token = await auth.GetAccessToken();
//eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJhS3VDVVBQV1RQd19vZDdpQUpzcDRPRkoxLXM2d0I5RVgzUDAyODhXRS1FIn0.eyJleHAiOjE2MjMzMDY0NzUsImlhdCI6MTYyMzMwNDY3NSwiYX....
```

You can see the API is similar to the Keycloak authenticator. At the end, you call `GetAccessToken()` to get the token.

Alternatively, for the ROPC flow create `AzureB2CAuthenticator` and set the `AddResourceOwnerPasswordCredentialFlowParameters` options:

```csharp
var auth = new AzureB2CAuthenticator(options =>
{
    options.AddResourceOwnerPasswordCredentialFlowParameters(
            new Uri("https://login.microsoftonline.com/<tenantID>/oauth2/v2.0/token"),
            "<clientId>"
            "<clientSecret>"
            new string[] { "https://graph.windows.net/.default" },
            "user",
            "pass");
});
            
var token = await auth.GetAccessToken();
//eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJhS3VDVVBQV1RQd19vZDdpQUpzcDRPRkoxLXM2d0I5RVgzUDAyODhXRS1FIn0.eyJleHAiOjE2MjMzMDY0NzUsImlhdCI6MTYyMzMwNDY3NSwiYX....
```

And that's it. Now you can use the token in your HTTP Client requests and start testing. I suggest you use [QAToolKit Http Tester library](https://github.com/qatoolkit/qatoolkit-engine-httptester-net).

_Please note that ROPC flow is mainly not recommended for a production environment with real users. There are more secure flows to use. I've decided to put ROPC to the `QAToolKit Auth library` because I think this flow might be handy for testing purposes, where we control the client (testing application) and the server (application we test)._