---
layout: post
title: Get Access Token (JWT) from Keycloak with QAToolKit
excerpt_separator: <!--more-->
author: Miha J.
tags: jwt, qatoolkit, C#, access token, azure b2c, identity server 4, keycloak
---
Our HTTP services are typically secured with some authentication. It can be `basic authentication`, `NTLM authentication`, `certificate authentication`, etc.

Today's prevalent means of authentication are access tokens (JWT), or bearer tokens, which are sent to our HTTP APIs in the `Authorization` header. When you use your application manually, you log in with your username and password to retrieve the token. But what when you do automatic testing without the UI?

QAToolKit to the rescue! There is a library in the QAToolKit repository called [QAToolKit Auth Library](https://github.com/qatoolkit/qatoolkit-auth-net). With it, you can get access tokens for your application from the three currently supported identity servers; `Keycloak`, `Identity Server 4`, and `Azure AD B2C`. Now I will show you how you can use it in [Keycloak](https://www.keycloak.org/).

We use `client credential flow` to retrieve the token. All you need to do is create the Keycloak authenticator, set the three parameters, and call `GetAccessToken()` method. This will return you the token as a string.

```csharp
var auth = new KeycloakAuthenticator(options =>
{
    options.AddClientCredentialFlowParameters(
                new Uri("https://my.keycloakserver.com/auth/realms/realmX/protocol/openid-connect/token"), 
                "my_client",
                "client_secret");
});

var token = await auth.GetAccessToken();
//eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJhS3VDVVBQV1RQd19vZDdpQUpzcDRPRkoxLXM2d0I5RVgzUDAyODhXRS1FIn0.eyJleHAiOjE2MjMzMDY0NzUsImlhdCI6MTYyMzMwNDY3NSwiYX....
```

Next, you can inject the token into your HTTP request header.

QAToolKit also supports exchanging the `client credential flow` token for the user token if your application demands it. All you need to do is to add the call `ExchangeForUserToken(email)` and pass in the user email address:

```csharp
var auth = new KeycloakAuthenticator(options =>
{
    options.AddClientCredentialFlowParameters(
                new Uri("https://my.keycloakserver.com/auth/realms/realmX/protocol/openid-connect/token"), 
                "my_client",
                "client_secret"); 
});

//Get client credentials flow access token
var token = await auth.GetAccessToken();
//Replace client credentials flow token for user access token
var userToken = await auth.ExchangeForUserToken("myuser@email.com");
//eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJhS3VDVVBQV1RQd19vZDdpQUpzcDRPRkoxLXM2d0I5RVgzUDAyODhXRS1FIn0.eyJleHAiOjE2MjMzMDY0NzUsImlhdCI6MTYyMzMwNDY3NSwiYX....
```

What you get is a user access token with user claims so that you can make actual tests on your HTTP API applications.

Next time we continue with Azure AD B2C and how you can retrieve the token from there.

Check out other [QAToolKit](https://qatoolkit.io/) tools here.