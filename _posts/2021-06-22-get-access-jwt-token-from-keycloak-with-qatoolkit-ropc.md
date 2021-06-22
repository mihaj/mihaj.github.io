---
layout: post
title: Get Access Token (JWT) from Keycloak with QAToolKit by ROPC
excerpt_separator: <!--more-->
author: Miha J.
tags: jwt, qatoolkit, C#, access token, keycloak, ROPC
---

I've introduced the [QAToolKit Auth library](https://github.com/qatoolkit/qatoolkit-auth-net) in the [previous post](https://www.mihajakovac.com/get-access-jwt-token-from-keycloak-with-qatoolkit/), where I wrote about retrieving the access token (JWT) from the Keycloak with `Client Credential flow`.

The QAToolKit Auth library supports another flow, and that is `Resource owner password credential` grant or ROPC. You can configure `KeycloakAuthenticator` differently like this:

```csharp
var auth = new KeycloakAuthenticator(options =>
{
    options.AddResourceOwnerPasswordCredentialFlowParameters(
                new Uri("https://my.keycloakserver.com/auth/realms/realmX/protocol/openid-connect/token"), 
                "keycloak_client",
                "keycloak_client_secret",
                "username",
                "password");
});

var token = await auth.GetAccessToken();
//eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJhS3VDVVBQV1RQd19vZDdpQUpzcDRPRkoxLXM2d0I5RVgzUDAyODhXRS1FIn0.eyJleHAiOjE2MjMzMDY0NzUsImlhdCI6MTYyMzMwNDY3NSwiYX....
```

Pass in the `Keycloak client ID`, `Keycloak client secret`, `username`, and `password`. For the token endpoint, you need to set it to your Keycloak instance and realm—referer to this [post](https://www.appsdeveloperblog.com/keycloak-requesting-token-with-password-grant/) for more information about Keycloak configuration.

After you call `GetAccessToken()` in the last line above, you'll receive the access token (JWT).

Please note that ROPC flow is mainly not recommended for a production environment with real users. There are more secure flows to use. I've decided to put ROPC to the `QAToolKit Auth library` because I think this flow might be handy for testing purposes, where we control the client (testing application) and the server (application we test).