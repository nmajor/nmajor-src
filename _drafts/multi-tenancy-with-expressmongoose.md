---
title: Multi Tenancy with Express/Mongoose
layout: post
date: 2018-09-28 00:00:00 +0000
hero: ''
tags: []

---
Multi tenent apps are apps where multiple users share the same database but their data is isolated from one another. This can basically describe almost any app with multiple users, but in my mind multi tenent apps are like having data isolation above the level of the user. For example you could have a data model called an `organization` and the the user can only see and interact with the data related to that organization.

Adding this kind of layer to apps is really useful and powerful and can really level up some apps.

The most important thing with multi tenent apps is that the data isolation must be perfect. If a user ever sees data not related to their tenent, it can result it a huge loss of customer trust especially these days when user data is such a hot issue.

We cant rely on developers writing queries and logic to keep this data isolation, so the best thing to do is to tackle this at the highest level possible.

The plan:

* **Express Middleware** - Add express middleware that will tell set the current tenent for the request.
* **Mongoose Wrapper** - Add a wrapper around Mongoose to make sure all queries are isolated to the current tenent
* 

By setting it up this way all mongoose queries and actions will automatically by isolated to their tenent, which makes it data leaking across tenents really unlikely.

### Express Middleware

One of the main tools we will be using is an amazing library called [node-continuation-local-storage](https://github.com/othiym23/node-continuation-local-storage "https://github.com/othiym23/node-continuation-local-storage").  Express is just a long chain of middleware functions. Functions that call functions that call functions. This library lets us set variables specific to a chain of functions. This means we can use it to set the `current_tenent` at the beginning of the request and then any function later in the chain, including any mongoose functions we call, will have access to the current tet. it lets us define variables for specific chains of functions. 