---
title: Get a Facebook Page Access Token That Never Expires
layout: post
date: 2018-03-21 00:00:00 +0000
hero: ''
tags: []
---
First we should talk about some definitions:

**short-lived user access token**: This is the regular user_access_token, and it typically lasts about 2 hours.

**long-lived user access token**: This user access token last about 60 days and you get it by extending a short-lived user access token

**page access token**: These are similar to user access tokens and are created using a user access token.

If you create a page access token using a short-lived user access token, it is also short-lived and expires in about 2 hours.

But if you create a page access token using a long-lived user access token, then it does not have an expiration date. It lasts until its access is revoked.

Bingo!

To get a user access token you first have to have a facebook app, and then the user (in this case, you) has to grant permissions to your app. Specifically your app needs the permission called `manage_pages`.

The easiest way to do this is to use [Facebook's Graph API Explorer](https://developers.facebook.com/tools/explorer).

Open up the page, then select your app from the dropdown:



Basically you have to first have a facebook app, and then you have to have the 



[https://developers.facebook.com/docs/facebook-login/access-tokens/expiration-and-extension](https://developers.facebook.com/docs/facebook-login/access-tokens/expiration-and-extension "https://developers.facebook.com/docs/facebook-login/access-tokens/expiration-and-extension")