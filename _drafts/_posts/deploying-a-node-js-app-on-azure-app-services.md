---
title: Deploying Node.js on Azure App Services
layout: post
date: 2018-02-14 00:00:00 +0000
tags:
- node
- azure
- production
hero: ''
---
### Create a deployment user

    az webapp deployment user set --user-name <username> --password <password>```

### Create a resource group

    az group create --name <my-resource-group> --location westeurope

### Create a service plan

List of service plans here:

[https://azure.microsoft.com/en-us/pricing/details/app-service/](https://azure.microsoft.com/en-us/pricing/details/app-service/ "https://azure.microsoft.com/en-us/pricing/details/app-service/")

    az appservice plan create --name <my-app-service-plan> --resource-group <my-resource-group> --sku FREE

### Create the app

    az webapp create --name <app-name> --resource-group <my-resource-group> --plan <my-app-service-plan>

### Configure git for deployment

    az webapp deployment source config-local-git --name <app-name> --resource-group <my-resource-group> --query url --output tsv