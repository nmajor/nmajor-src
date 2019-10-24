---
layout: post
title: How to Make JWT Authentication Not Suck
hero: ''
tags: []
date: 

---
This is not an introduction to JSON Web Tokens. There are plenty of those on the internet already. I'm trying to outline and compile my thoughts and research about when to implement JWT and how to not do it poorly.

If you want a good intro check out [this post](https://jwt.io/introduction/ "https://jwt.io/introduction/") and [this post](https://blog.angular-university.io/angular-jwt/ "https://blog.angular-university.io/angular-jwt/").

First of all, why are JWT or JSON Web Token useful and so widely used. 

* **Stateless** - They don't require storing sessions in a database.
* **Portable** - They can easily be used with multiple different backends and services.
* **No cookies** - They can easily be used as bearer tokens by multiple different kinds of clients (mobile, browser, etc...)
* **Verbose** - They include useful information in the token itself like user roles and other claims about the user.

Is there any reason NOT to use JWT.

JWTs are interesting in that they are widely used and also widely criticized. Most criticisms of JWT fall into 2 categories:

* **Overkill** - There are a lot of moving parts with JWT. Its probably overkill for many useless.
* **Library/Implementation Vulnerabilities** - Criticizing vulnerabilities in particular JWT libraries or implementations.
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
* I am willing to keep my implementation up to date.

If you are willing to take the time to understand JWT, then there are some resources to get you started.

* Intro to JWT - [https://jwt.io/introduction/](https://jwt.io/introduction/ "https://jwt.io/introduction/")
* The Complete Guide to JSON Web Tokens - [https://blog.angular-university.io/angular-jwt/](https://blog.angular-university.io/angular-jwt/ "https://blog.angular-university.io/angular-jwt/")

### Library/Implementation Vulnerabilities

It seems like this is an easy one to address. Just use a good library and good implementation. lol.

##### Library

But seriously JWT as a concept is supported strongly by the company Auth.io and they curate a list of approved JWT libraries for many different libraries and approved hashing algorithms.

That [list of approved libraries can be found here](https://jwt.io/#libraries "https://jwt.io/#libraries").

Since The other important part of this concern is keeping the library up to date.However this is an issue with all opensource projects.

If security is a primary concern, then you should already have a strategy to keep notified of the vulnerabilities in the libraries you are using.

If you work with Node.js then the `npm audit` command should be an important part of your deployment workflow. Of course its limited to known and reported library vulnerabilities, but its much better than doing nothing.

More info on [npm audit](https://blog.npmjs.org/post/173719309445/npm-audit-identify-and-fix-insecure "https://blog.npmjs.org/post/173719309445/npm-audit-identify-and-fix-insecure") and [npm security advisories]().

If you're working in a language other than Node.js then you'll need to have your own strategy of staying appraised of library vulnerabilities.

##### Implementation

Good implementation comes from understanding. I think its safe to say that blindly copy/pasting solutions from stack overflow really increases your likelihood of having a poor implementation.

This is authentication we're talking about. And given how catastrophic the consequences can be, make sure you educate yourself and tread carefully when building your implementation.

Avoid blind copy/pasta unless you understand what its doing. Also personally I like to avoid using big black box libraries and build the minimum functional system.

Here are some resources to help:

* JWT Best Current Practices - [https://auth0.com/blog/a-look-at-the-latest-draft-for-jwt-bcp/](https://auth0.com/blog/a-look-at-the-latest-draft-for-jwt-bcp/ "https://auth0.com/blog/a-look-at-the-latest-draft-for-jwt-bcp/")
* Critical vulnerabilities in JSON Web Token libraries - [https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries/](https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries/ "https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries/")

### Stateless Auth Downsides

Here is where some personal preference comes in. One of the biggest downsides to stateless authentication is that there is no way of invalidating the tokens once they are issued. This is a potential security issue as well as an inconvenience.

But I think this can be solved by using a stateful refresh token.

This does add more moving parts to the authentication machine, but it solves the important issue of revoking token access.

There are many ways to implement this. For me, at this time, I prefer this:

* Refresh tokens are stored in the database of an authentication service. One user can have many refresh tokens.
* Refresh tokens can only be issued with a full authentication process like sending a valid password, single sign on flow, etc....
* The response to a successful strong authentication includes a JWT (that includes the expiration time inside the token) and a refresh token.
* The client is responsible for keeping their JWT up to date by monitoring the expiration time and refreshing using the refresh token.

### Alternatives to JWT

* PASETO - [https://paseto.io](https://paseto.io "https://paseto.io")
  * Main stateless alternative to JWT it has restrictions on algorithms and other things to help prevent developers from making common mistakes.
  * 