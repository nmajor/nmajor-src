---
title: Express.js With ES6 and Babel
layout: post
date: 2018-02-16 00:00:00 +0000
tags:
- express
- node
- babel
- es6
hero: ''
---
Express.js doesn't come with es6 out of the box. But its actually not that hard to add.

This is the best way I have found to add ES6 support to an express project. I'm going to go over how to easily add ES6 support to your project using the [babel-cli](https://www.npmjs.com/package/babel-cli "babel-cli").

The `babel-cli` gives us a command line tool called `babel-node` that wraps around node and converts our ES6 code to basic javascript that node can read.

**However it is not recommended to use babel-node in production**

So I will also show you how to prepare your Express.js app for production with a build step.

  
  
  
  
  
The `babel-cli` gives us access to the `babel-node` command. But `babel-node` is NOT recommended for production use, I will also show you how to build your 