---
layout: post
title: Azure B2C Login throws errors AADB2C99059 or AADB2C90079
excerpt_separator: <!--more-->
author: Miha J.
tags: azure, azure b2c, authentication, AADB2C99059, AADB2C90079 
---

I am using Azure B2C as an identity server in one of my applications. Setting it up is easy, and you get a modern authentication and authorization infrastructure. We use it with an application that has a .NET back-end application and a .NET MVC front-end. I am not saying it's the best option out there.

Because Azure AD B2C [retired](https://azure.microsoft.com/en-us/updates/update-apps-using-azure-ad-b2c-to-new-redirect-b2clogincom/) `login.microsoftonline.com` last month our MVC app did not authenticate properly anymore. We needed to update it.

When trying login to our application, we got this error:

```text
AADB2C90079: Clients must send a client_secret when redeeming a confidential grant.
```

A quick google search pointed to [Stackoverflow post](https://stackoverflow.com/questions/59163544/azure-ad-b2c-clients-must-send-a-client-secret-when-redeeming-a-confidential-gr).

I've changed the application type from WEB to SPA and then got another error:

```text
AADB2C99059: The supplied request must present a code_challenge.
```

I opened up the front-end MVC application code and changed `OpenIdConnectOptions` by adding the following:

```c#
public void Configure(string name, OpenIdConnectOptions options)
{
    ...
    
    options.ResponseType = OpenIdConnectResponseType.Code;
    
    ...
}
```

Now the app worked because the `code_challenge` parameter was sent down. It's a quick patch, but I discovered that the application used [Implicit flow](https://learn.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-implicit-grant-flow), which is not recommended anymore.

I've replaced it with [Authorization code flow](https://learn.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-auth-code-flow) by removing event `OnAuthorizationCodeReceived` from the `OpenIdConnectEvents`. The `OnAuthorizationCodeReceived` code used client credential flow to obtain the tokens, and I decided to remove that and do the `Authorization code flow`.

Additionally, I've enabled [PKCE](https://www.rfc-editor.org/rfc/rfc7636) (Proof Key for Code Exchange)

Before:

```c#
public void Configure(string name, OpenIdConnectOptions options)
{
    options.ClientId = _azureOptions.ClientId;
    options.Authority = $"{_azureOptions.Instance}/{_azureOptions.Domain}/{_azureOptions.SignUpSignInPolicyId}/v2.0";
    options.UseTokenLifetime = true;
    options.CallbackPath = $"{_azureOptions.CallbackPath}";
    options.SaveTokens = true;

    options.TokenValidationParameters = new TokenValidationParameters { NameClaimType = "name" };

    options.Events = new OpenIdConnectEvents
    {
        OnRedirectToIdentityProvider = OnRedirectToIdentityProvider,
        OnRemoteFailure = OnRemoteFailure,
        OnAuthorizationCodeReceived = OnAuthorizationCodeReceived
    };
}
```

After:

```c#
public void Configure(string name, OpenIdConnectOptions options)
{
    options.ClientId = _azureOptions.ClientId;
    options.Authority = $"{_azureOptions.Instance}/{_azureOptions.Domain}/{_azureOptions.SignUpSignInPolicyId}/v2.0";
    options.UseTokenLifetime = true;
    options.CallbackPath = $"{_azureOptions.CallbackPath}";
    options.SaveTokens = true;
    options.UsePkce = true;
    options.ResponseType = OpenIdConnectResponseType.Code;
                
    options.TokenValidationParameters = new TokenValidationParameters { NameClaimType = "name" };

    options.Events = new OpenIdConnectEvents
    {
        OnRedirectToIdentityProvider = OnRedirectToIdentityProvider,
        OnRemoteFailure = OnRemoteFailure
    };
}
```

Less code with better security! :)