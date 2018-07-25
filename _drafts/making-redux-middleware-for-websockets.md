---
title: Redux Middleware for Websockets and Profit
layout: post
date: 2018-07-25 00:00:00 +0000
hero: ''
tags: []

---
Middleware is one of the most useful and powerful features of redux. If you're unfamiliar with redux middleware, basically it is a way to insert extra behavior into dispatched redux actions.

In this post I'll show you 2 different websocket redux middleware functions I use to manage event listening and subscribing to different rooms, etc...

{: .lead}  
<!–-break-–>

I'll show you how to set up middleware for both socket.io and Action Cable.

If you are unfamiliar with redux middleware, check out the \[documentation here\]([https://redux.js.org/advanced/middleware](https://redux.js.org/advanced/middleware "https://redux.js.org/advanced/middleware")). This approach is inspired by reading through the source of \[this example redux app\]([https://github.com/erikras/react-redux-universal-hot-example](https://github.com/erikras/react-redux-universal-hot-example "https://github.com/erikras/react-redux-universal-hot-example")), specifically \[this\]([https://github.com/erikras/react-redux-universal-hot-example/blob/master/src/redux/middleware/clientMiddleware.js](https://github.com/erikras/react-redux-universal-hot-example/blob/master/src/redux/middleware/clientMiddleware.js "https://github.com/erikras/react-redux-universal-hot-example/blob/master/src/redux/middleware/clientMiddleware.js")) middleware, so you may also want to check that out as well.

Basically redux action, by default, looks like this:

    { type: 'MY_ACTION_TYPE' ...some_data }

Redux actions have 1 required attribute, `type`. Anything else is just extra and is usually meant to be used by the reducer to mutate the state.

By using redux middleware we can define our own action patterns and structures. Then when any action is dispatched we can check if the action matches a specific pattern and have it do different things.

## Redux Middleware for Action Cable

Anyways, I dont want to spend too much time on this, but I was so impressed with how powerful this is that when it came time to add websockets to my app I 