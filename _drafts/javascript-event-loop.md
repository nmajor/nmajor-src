---
layout: post
title: Javascript Event Loop Basics
hero: "/uploads/2019/10/08/Event Loop basics.png"
tags:
- javascript
- node
date: 

---
#### The event loop is not technically part of javascript

The first important point about the Javascript Event Loop is that, although its an important part of javascript, its technically not part of javascript. The event loop comes from the runtime environment, so depending on your environment there will be small differences in the event loop. Chrome has a slightly different event loop than Node for example.

#### Javascript is single threaded and blocking. The event loop makes it non-blocking and asynchronous.

You've probably heard of the V8 javascript engine. It is the javascript engine used by Chrome and Node. The javascript V8 engine is single threaded. The event loop manages a stack and task queue and sometimes even worker threads to make javascript behave in a non-blocking async way for certain functions.

  
  
Javascript runs on a single thread which means it can only do 1 thing at a time. So how is is asynchronous?

Anyone who has used javascript knows that some things happen synchronously one line at a time while other things are async and require callbacks.

The Event Loop is a non-blocking runtime component loop that makes javascript asynchronous. However this is a bit misleading because the Javasc