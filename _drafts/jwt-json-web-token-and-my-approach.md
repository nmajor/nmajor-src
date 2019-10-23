---
layout: post
title: JWT (JSON Web Token) and My Approach
hero: ''
tags: []
date: 

---
First of all, why are JWT or JSON Web Token useful and so widely used.

* **Stateless** - They don't require storing sessions in a database.
* **Portable** - They can easily be used with multiple different backends and services.
* **No cookies** - They can easily be used as bearer tokens by multiple different kinds of clients (mobile, browser, etc...)
* **Verbose** - They include useful information in the token itself like user roles and other claims about the user.

Is there any reason NOT to use JWT. 

JWT or JSON Web Tokens are interesting in that they are widely used and also widely criticized. As mentioned in this [hackernews thread](https://news.ycombinator.com/item?id=13865459 "https://news.ycombinator.com/item?id=13865459"), most criticisms fall into 2 categories:

* **Overkill** - There are a lot of moving parts with JWT. Its probably overkill for many useless.
* **Library Vulnerabilities** - Criticizing vulnerabilities in particular JWT libraries.
* **Implementation Vulnerabilities** - Criticizing vulnerabilities in particular implementations.
* **Stateless Auth Downsides** - Generally criticizing the practice of using any "stateless" client tokens. Because there's no great way to revoke them early while remaining stateless, etc.

I'm going to try to give my best shot at addressing all of these issues and outline an approach that might work for some people.

At the bottom of this post I've listed some alternative API authentication methods to to explore.

### Overkill

First you should decide if your need/want the benefits of JWT.

Because JWT has so many moving parts (less than other forms of authentication but more than others), in my opinion, you should only do it if you need some of the unique benefits of JWT.

Be sure to checkout some of the alternatives to JWT that I've listed below.

I would say the following 2 things describe you then JWT might be worth implementing:

* I have multiple services/backends share authentication.
* I have non-browser clients that use my service.
* I am willing to take the time to understand JWT before implementing it.

If you are willing to take the time to understand JWT, then there are some resources to get you started. 

* Intro to JWT - [https://jwt.io/introduction/](https://jwt.io/introduction/ "https://jwt.io/introduction/")
* Critical vulnerabilities in JSON Web Token libraries - [https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries/](https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries/ "https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries/")

### Library Vulnerabilities

It seems like this is an easy one to address. Just use a good library. lol.

But seriously JWT as a concept is supported strongly by the company Auth.io and they curate a list of approved JWT libraries for many different libraries and approved hashing algorithms.

That [list of approved libraries can be found here](https://jwt.io/#libraries "https://jwt.io/#libraries").

The first point is only resolved by using a solid implementation and algorithm.

The second point I think can be solved with a refresh token strategy. You can generate refresh tokens for each user and store/manage them in the database of an authentication service. Then whenever the client gets a new JWT from the authentication server it also will get the expiration time of the token as well as a refresh token. The client is then responsible for refreshing the tokens with the authentication service before sending to any api. This allows for easy token invalidation by invalidating the refresh token. Of course there is a compromise with 

I've settled on a kind of hybrid solution that I think addresses both of these points.

Before you dive into securing an API with JWT its important to address both of these issues. I recommend anyone interested in implementing JWT authentication first read some of these resources:

* Critical vulnerabilities in JSON Web Token libraries - [https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries/](https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries/ "https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries/")

In addressing first point about vulnerabilities in different JWT libraries 