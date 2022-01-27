---
layout: post
title: Set Environment variables to Azure Container App
excerpt_separator: <!--more-->
author: Miha J.
tags: azure, docker, container apps, net6
---

To run your Linux containers in Kubernetes in Azure as lean as possible, check out a new service [Azure Container Apps](https://azure.microsoft.com/en-us/services/container-apps/#overview). It is still in the preview and a bit quirky, but it worked for me in the end! I do not run any production workload on it, but I tried to run a .NET 6 Web API application for a DEMO and hit a wall setting the environment variables. After some reading and playing with CLI, I've compiled this short tutorial below. Stick around :).

There are many excellent guides on creating and deploying Linux containers to the Azure Container Apps, and one of them is [here](https://techcommunity.microsoft.com/t5/itops-talk-blog/azure-container-apps-ci-cd-deployments-video-demo/ba-p/3056192).

Once you log in to the Azure portal and go to your Container app, you only see the secrets menu and no configuration for environment variables (as shown in the screenshot below). Not to worry, we can set them through the `az` CLI. Once the service is out of preview, I expect the Azure team to implement the UI capability.

![Azure container app secrets](images/azure_container_app_secrets.png)

My DEMO application is a .NET 6 Web API and needs a database connection string, so I set a secret with `database-connectionstring` through the Azure Portal.

![Add a secret](images/azure_container_app_secret_add.png)

Also the Ingress is set like this:

![Azure container app ingress](images/azure_container_app_ingress.png)

Next, we need to [install](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) `az` CLI. I've compiled the steps in the script below.

Alternatively, you can use in-browser CLI, by clicking on the button in the toolbar: 

![Azure CLI](images/azure_container_app_cli.png)

Once you have your `az` CLI ready, go through the script below and set environment variables.

```powershell
# Install `containerapp` module: https://docs.microsoft.com/en-us/azure/container-apps/get-started?tabs=bash. Current version:
az extension add --source https://workerappscliextension.blob.core.windows.net/azure-cli-extension/containerapp-0.2.0-py2.py3-none-any.whl
# Login to azure if not logged in yet
az login
# List your container apps and check your deployment configuration
az containerapp list --resource-group MyResourceGroup
# Set the app to listen on port 80 (or whatever you have set up)
az containerapp update -v 'ASPNETCORE_URLS=http://+:80' --resource-group=MyResourceGroup --name my-container-app
az containerapp update -v 'urls=http://+:80' --resource-group=MyResourceGroup --name my-container-app
# use __ for nested configuration
az containerapp update -v 'MyService__Setting1=1' --resource-group=MyResourceGroup --name my-container-app
# Link more complex variables through secrets. You first need to create a secret with a name 'database-connectionstring' and set it to the environment variable 'MyService__Database__ConnectionString'
az containerapp update -v 'MyService__Database__ConnectionString=secretref:database-connectionstring' --resource-group=MyResourceGroup --name my-container-app
```

Azure Container Apps is a good service that abstracts all the complexity away from us to focus more on developing our applications. Let's see what is next in coming months. :)