---
layout: post
title: Javascript Event Loop Basics
hero: ''
tags:
- javascript
date: 

---
Javascript runs on a single thread which means it can only do 1 thing at a time. So how is is asynchronous? 

Anyone who has used javascript knows that some things happen synchronously one line at a time while other things are async and require callbacks. 

The Event Loop is a non-blocking runtime component loop that makes javascript asynchronous. However this is a bit misleading because the Javasc