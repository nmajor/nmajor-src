---
title: Using Socket.io with Redux - Websocket Redux Middleware
layout: post
date: 2018-08-21 13:59:28 +0100
hero: "/uploads/2018/07/25/Action Cable (1).png"
tags:
- javascript
- react
- redux
- websockets
- socket.io

---
This article uses socket.io as the websocket library. If you want to see a version of this article using Rails' ActionCable library, [click here](/posts/making-redux-middleware-for-websockets "/posts/making-redux-middleware-for-websockets")

***

Middleware is one of the most powerful and useful features of redux. If you're unfamiliar with redux middleware, it is a way to insert extra behavior into dispatched redux actions.

Today we're going to use it to make a clean and powerful way to manage our socket.io events. Basically we want to subscribe to specific events and then dispatch specific redux actions anytime those events are broadcast to us from the socket.io server.

{: .lead}  
<!–-break-–>

Check out the [redux middleware documentation here](https://redux.js.org/advanced/middleware "https://redux.js.org/advanced/middleware") The code below is inspired by reading through the source of [this example redux app](https://github.com/erikras/react-redux-universal-hot-example "https://github.com/erikras/react-redux-universal-hot-example"), specifically [this middleware](https://github.com/erikras/react-redux-universal-hot-example/blob/master/src/redux/middleware/clientMiddleware.js "https://github.com/erikras/react-redux-universal-hot-example/blob/master/src/redux/middleware/clientMiddleware.js"), so you may also want to check that out as well.

### Typical Redux Actions

Redux actions, by default, looks like this:

    { type: 'MY_ACTION_TYPE' ...some_data }

Redux actions have 1 required attribute, `type`. Anything else is just extra and is usually meant to be used by the reducer to mutate the state.

By using redux middleware we can define our own action patterns and structures. The middleware will check if the action has other specific attributes and handle that action differently than the others. That way all we have to do to trigger some custom redux behavior is dispatch an action with our specific attributes and it will automatically be handled differently.

Specifically we are going to create a new kind of action that will start listening for specific socket.io events and dispatch a new redux action anytime that event comes through from the server. We'll also be able to stop listening for any specific event.

### Middleware Function

First lets make a middleware function, you'll want to export this function from a file. I called my file \`socketMiddleware.js\`:

    import io from 'socket.io-client';
    
    export default function socketMiddleware() {
      const socket = io();
    
      return ({ dispatch }) => next => (action) => {
        if (typeof action === 'function') {
          return next(action);
        }
    
        const {
          event,
          leave,
          handle,
          ...rest
        } = action;
    
        if (!event) {
          return next(action);
        }
    
        if (leave) {
          socket.removeListener(event);
        }
    
        let handleEvent = handle;
        if (typeof handleEvent === 'string') {
          handleEvent = result => dispatch({ type: handle, result, ...rest });
        }
        return socket.on(event, handleEvent);
      };
    }

Lets break down this code:

Basically we first skip our middleware if the action is a function or if there is no `event` attribute in our action.

Then if there is a `leave` attribute, then we remove the event listener

Else we create a new event listener.

But you will notice that we are doing some quick logic to check if the `handle` attribute is a string. And if it is, we are changing its value to a function that dispatches a new action with the received data. So basically our `received` attribute can take both an action type string or an actual function. This gives us an extra level of control over how we handle the data coming in from the socket.io server.

And any data that is sent from the socket.io server is included in the `result` attribute of the action. So the reducer can have easy access to any data the server sent. Also any extra action attributes (`...rest`) are just passed directly through to the dispatched action.

Also notice that the function that we are exporting is returning another function. This is an example of a [Higher-order function](https://en.wikipedia.org/wiki/Higher-order_function "https://en.wikipedia.org/wiki/Higher-order_function") and is a very useful pattern in Javascript. We will execute the outside function when we apply the middleware to redux which will create the socket.io connection only once and give the inner function access to that connection going forward.

### Add Middleware to Redux

Then we have to apply our new middlware. Check the [redux documentation](https://redux.js.org/advanced/middleware#attempt-6-naively-applying-the-middleware "https://redux.js.org/advanced/middleware#attempt-6-naively-applying-the-middleware") for how to do this. But you will probably need to do something like this when setting up your store:

    import { createStore, applyMiddleware } from 'redux';
    import socketMiddleware from './middleware/socketMiddleware';
    import rootReducer from './reducers/index';
    
    const store = createStore(
      rootReducer,
      applyMiddleware(socketMiddleware())
    );

Since other file is actually exporting a higher-order function, don't forget to execute the socketMiddleware function when applying it to redux. This will also create the socket.io connection only once when the store loads so we don't have to worry about creating a new connection every time.

### Our New Action Creators

We now have access to a new type of action that has new required attributes. If we dispatch an action with a `event` attribute it will trigger our socket middleware.

Here are some example action creators using our new middleware.

    export function subscribeMessages() {
      return {
        event: 'message',
        handle: NEW_MESSAGE,
      }
    }
    
    export function unsubscribeMessages() {
      return {
        event: 'message',
        leave: true,
      }
    }
    
    // Action creator with received function:
    export function subscribeConversation() {
      return dispatch => dispatch({
        event: 'message',
        handle: data => dispatch({
          type: NEW_MESSAGE,
          payload: data.message,
        }),
      });
    }

You'll notice these actions don't even have the required `type` attribute, this is because when they are dispatched we hijack the action and do out own thing, so these particular actions never makes it to the reducer.

Instead we have an `event`, and `handle` attribute required to start listening for an event, and `event` and `leave`, attributes to stop listening.

The important part here is that `handle` is either an action string to dispatch when new data comes in, or a function to run when new data comes in, but since that function can be another call to dispatch, we have the convenience of a useful default and the flexibility to do whatever we want.

### Conclusion

I'm now a huge fan of redux middleware, I love how it cleans up and simplifies doing complex repetitive things in our action creators.

Now we've set this up we can subscribe and unsubscribe to socket.io events really easily just by dispatching one of our new action creators.