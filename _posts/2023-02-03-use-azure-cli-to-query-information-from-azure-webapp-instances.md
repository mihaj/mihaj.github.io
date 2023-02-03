---
layout: post
title: Use Azure CLI (az) to query information from Azure Web App instances
excerpt_separator: <!--more-->
author: Miha J.
tags: azure, azure web app, azure cli, az
---

I've been playing more with Azure CLI (az) lately and discovering new useful commands. From deploying new Docker images to Azure Web App service to retrieving the status of the apps.

I want to show you how you easily list the Azure Web App instances information in JSON format with Azure CLI.

First, you must download the [AZ CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli).

Next, you need to log in to Azure with AZ CLI.

```powershell
az login
```
Let's first list all the Azure Web App instances under our Azure tenant. Then you can start querying the Azure API with AZ CLI. You can achieve this by:

```powershell
az webapp list
```
We get back a lot of information for every instance. It's too much useless information if you only need a status report. Let's modify the command to introduce querying or filtering. There are five parameters I like to see in the response.

Host name of the application,
state of the app (is it running?),
the version of running Docker image,
the parent resource group of the application.

```powershell
az webapp list --query "[].{hostName: defaultHostName, state: state, image: siteConfig.linuxFxVersion, resourceGroup: resourceGroup}"
```
You'll receive a smaller portion of the request data when you run this.

```json
{
    "hostName": "myapp.azurewebsites.net",
    "image": "DOCKER|myapp.azurecr.io/myapp:latest",
    "resourceGroup": "MyApp",
    "usage": "Normal"
}
```

You can expand the querying filter in the AZ CLI command by running the `az webapp list` and choosing the values you want in your response.

Here is a shortened version of the JSON response that you get back when running `az webapp list`:

```json
...
   "enabledHostNames": [
      "myapp.com",
      "yourapp.com",
    ],
    "hostNameSslStates": [
      {
        "hostType": "Standard",
...
      },
      {
        "hostType": "Standard",
...
      }
    ],
...
    "siteConfig": {
      "acrUseManagedIdentityCreds": false,
      "acrUserManagedIdentityId": null,
      "alwaysOn": true,
...
```
To query the first value of `enabledHostNames`, `hostType` of the second `hostNameSslStates` object, and `alwaysOn` from `siteConfig`, we would write this:

```powershell
az webapp list --query "[].{firstHostName: enabled, secondHostType: hostNameSslStates[1].hostType, alwaysOn: siteConfig.alwaysOn}"
```

You should receive something similar back:

```json
{
    "alwaysOn": true,
    "firstHostName": true,
    "secondHostType": "Standard"
}
```

That's it! Another way to quickly get the status of your applications. Imagine linking the resulting JSON to a UI dashboard like Grafana!