---
title: Get a Facebook Page Access Token That Never Expires
layout: post
date: 2018-03-21 00:00:00 +0000
hero: "/uploads/2018/03/21/So, let's take it to the streets and celebrate our freedom!.png"
tags:
- facebook
---
With the facebook API, page tokens can often be used in place of user tokens. This is very useful because if you are doing something with a business, its not ideal to have to use your own user token for everything. However, if you don't want to have to recreate a new token all the time, you have to go through the process of getting a long-lived page access token.

{: .lead}  
<!–-break-–>

First we should talk about some definitions:

**short-lived user access token**: This is the regular user_access_token, and it typically lasts about 2 hours.

**long-lived user access token**: This user access token last about 60 days and you get it by extending a short-lived user access token

**page access token**: These are similar to user access tokens and are created using a user access token.

If you create a page access token using a short-lived user access token, it is also short-lived and expires in about 2 hours.

But if you create a page access token using a long-lived user access token, then it does not have an expiration date. It lasts until its access is revoked.

Bingo!

To get a user access token you first have to have a facebook app, and then the user (in this case, you) has to grant permissions to your app. Specifically your app needs the permission called `manage_pages`.

The easiest way to do this is to use [Facebook's Graph API Explorer](https://developers.facebook.com/tools/explorer).

Open up the page, then select your app Application dropdown and select "Get User Access Token" from the other dropdown:

![](/uploads/2018/03/21/Graph_API_Explorer_-_Facebook_for_Developers-1.png)

Then in the popup, make sure you check the box for `manage_pages` and click "Get Access Token"

![](/uploads/2018/03/21/Graph_API_Explorer_-_Facebook_for_Developers2.png)

Then your access token will be populated in the Access Token field:

![](/uploads/2018/03/21/Graph_API_Explorer_-_Facebook_for_Developers3.png)

Ok, then we need to exchange that short-lived token for a long-lived token, and we can do it with this same Facebook Graph API Explorer tool.

We need to make a GET request to the Facebook Graph API to the path `/oauth/access_token` with the following parameters:

    /oauth/access_token?  
        grant_type=fb_exchange_token&           
        client_id={app-id}&
        client_secret={app-secret}&
        fb_exchange_token={short-lived-token-we-just-made} 

So make a string that looks like this:

    /oauth/access_token?grant_type=fb_exchange_token&client_id=9382767696389008&client_secret=1aac234d48ce90uu57c1b579faa92f00ufa&fb_exchange_token=EAAcZBbBDPXSwBACfeRskLz5LJJCigZBZBDyr4pmZCcVsrpFaoDUDSXthSDzdq4bqO75iEE17TBbpxIyQ1eTR3OIfGq2KPZBloF4xJ0I1GU1hxOnCGJmZBVg0DXU4IrDgcURm4ceKE4M1ZAFdGNmt5y2shsETLxAEKwZD

And paste it into here and you'll get your long-lived access token in the response:

![](/uploads/2018/03/21/Graph_API_Explorer_-_Facebook_for_Developers4-1.png)

Now that you have your long-lived user access token, we can exchange it by making a GET request in the Facebook Graph API Explorer to get the long-lived page access token.

You need to take that long-lived user access token and paste it into the Access Token field then put this in the path field:

    {fb-page-id}]/?fields=access_token

Then you will get your long-lived page access token in the response!

![](/uploads/2018/03/21/Graph_API_Explorer_-_Facebook_for_Developers5.png)

Facebook Documentation:

* Access Tokens: [https://developers.facebook.com/docs/facebook-login/access-tokens](https://developers.facebook.com/docs/facebook-login/access-tokens "https://developers.facebook.com/docs/facebook-login/access-tokens")
* Extending Tokens: [https://developers.facebook.com/docs/facebook-login/access-tokens/expiration-and-extension](https://developers.facebook.com/docs/facebook-login/access-tokens/expiration-and-extension "https://developers.facebook.com/docs/facebook-login/access-tokens/expiration-and-extension")