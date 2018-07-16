---
title: Deploying Node.js on Azure App Services
layout: post
date: 2018-02-14 00:00:00 +0000
tags:
- node
- azure
- production
hero: "/uploads/2018/07/16/Deploying.png"

---
I know, I know, I know, Microsoft is not the company you think of when its time to figure out where to host your Node.js app, but Azure has a surprisingly robust suite of services now. And deploying to [Azure App Services](https://azure.microsoft.com/en-us/services/app-service/ "Azure App Service") was easier than I expected.

In this post I'm going to go over how to deploy a Node.js app to Azure App Services and also mention some of the things I learned from working with Azure.

# Your Azure Account

### Azure account

First of all, you need to make sure you have an Azure account. If you don't have one you can sign up for Azure [here](https://azure.microsoft.com/free/ "Azure Signup").

### Azure Subscription

Azure requires you to have something called a subscription before you can start using Azure services. This subscription is basically a billing program.

Double check that you have a subscription by going to this page in your Azure portal:

[https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade "https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade")

If you don't see a subscription listed, then add one.

If you have just signed up you'll notice that one of the subscription options is "Free Trial" so you can use that and then later change to a "Pay as You Go" option later if you run out of free resources.

### Azure Command Line Tool

We are going to be doing this entire setup using the Azure command line tool.

You can download and install the command line tool by following the \[instructions here\]([https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest "https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest"))

Once you have that installed and working, we are ready to get started deploying the app.

### Create a deployment user

This deployment user will be used when deploying your code. Basically we are going to configure the app to be deployed with git so when you do a `git push azure master` it will ask you for the password that you set here.

Run this in the command line:

    az webapp deployment user set --user-name <username> --password <password>

### Create a resource group

Resource groups are basically just a way to group together resources so you can easily monitor them together, track costs, and stuff.

Replace `<my-resource-group>` with a resource group name you want.

    az group create --name <my-resource-group> --location westeurope

### Create a service plan

This is where you will set the how many resources your app will use.

The command below will create a `FREE` tier app that has about 1 GB of RAM. You could also set the `--sku` tag to `BASIC` or `SHARED` or any of the other \[service plans here\]([https://azure.microsoft.com/en-us/pricing/details/app-service/](https://azure.microsoft.com/en-us/pricing/details/app-service/ "https://azure.microsoft.com/en-us/pricing/details/app-service/")).

Also be sure to set your `<my-app-service-plan>` and also make sure you set `<my-resource-group>` to your resource group name from earlier.

    az appservice plan create --name <my-app-service-plan> --resource-group <my-resource-group> --sku FREE

### Create the app

Now we actually create the app, you have to give your app an `<app-name>` and then make sure you paste in all the other options from things you've created previously.

    az webapp create --name <app-name> --resource-group <my-resource-group> --plan <my-app-service-plan>

### Configure git for deployment

This command sill set up you app to be deployed using git. The response you get from this command will be a git url you can use as the git remote to deploy.

    az webapp deployment source config-local-git --name <app-name> --resource-group <my-resource-group> --query url --output tsv

### Configure startup script

If you are coding in ES6 you probably have a build step. For that you may want to set up a custom deploy script. \[Here is the documentation for how to do that\]([https://github.com/projectkudu/kudu/wiki/Custom-Deployment-Script](https://github.com/projectkudu/kudu/wiki/Custom-Deployment-Script "https://github.com/projectkudu/kudu/wiki/Custom-Deployment-Script")).

However I've noticed a bug here. Sometimes the order of commands in the deploy script appears to not be honored. According to \[this github issue\]([https://github.com/projectkudu/kudu/issues/2699](https://github.com/projectkudu/kudu/issues/2699 "https://github.com/projectkudu/kudu/issues/2699")), it may be related to the Always On setting in your app. But regardless the app tries to start up before the my deploy script is finished, which means it tries to run code that isnt built from webpacker yet.

So, in an effort to find a quick fix I just committed my built files which sucks. I'm still trying to figure out a better workaround here because there hasn't been a lot of movement on that particular github issue.