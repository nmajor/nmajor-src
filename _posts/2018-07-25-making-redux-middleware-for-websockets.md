---
title: Using Action Cable with Redux - Websocket Redux Middleware
layout: post
date: 2018-07-25 00:00:00 +0000
hero: "/uploads/2018/07/25/Action Cable.png"
tags:
- redux
- javascript
- websockets
- react
- rails
- actioncable

---
This article uses ActionCable as the websocket library. If you want to see a version of this article using socket.io, [click here.](/posts/using-socket-io-with-redux-websocket-redux-middleware "/posts/using-socket-io-with-redux-websocket-redux-middleware")

***

Middleware is one of the most powerful and useful features of redux. If you're unfamiliar with redux middleware, it is a way to insert extra behavior into dispatched redux actions.

Today we're going to use it to make a clean and powerful way to manage our subscriptions to different Action Cable channels+rooms. This also means taking the data sent to use through action cable and dispatching the appropriate redux actions to mutate the state.

{: .lead}  
<!–-break-–>

If you are unfamiliar with redux middleware, check out the [documentation here](https://redux.js.org/advanced/middleware "https://redux.js.org/advanced/middleware"). The code below is inspired by reading through the source of [this example redux app](https://github.com/erikras/react-redux-universal-hot-example "https://github.com/erikras/react-redux-universal-hot-example"), specifically [this middleware](https://github.com/erikras/react-redux-universal-hot-example/blob/master/src/redux/middleware/clientMiddleware.js "https://github.com/erikras/react-redux-universal-hot-example/blob/master/src/redux/middleware/clientMiddleware.js"), so you may also want to check that out as well.

### Typical Redux Actions

Basically redux action, by default, looks like this:

```javascript
{ type: 'MY_ACTION_TYPE', other: 'data' }
```
Redux actions have 1 required attribute, `type`. Anything else is just extra and is usually meant to be used by the reducer to mutate the state.

By using redux middleware we can define our own action patterns and structures. The middleware will check if the action has other specific attributes and handle that action differently than the others. That way all we have to do to trigger some custom redux behavior is dispatch an action with our specific attributes and it will automatically be handled differently.

Specifically we are going to create a new kind of action that will subscribe or unsubscribe to specific Action Cable channels+rooms.

### Middleware Function

First lets make a middleware function, you'll want to export this function from a file. I called my file \`cableMiddleware.js\`:

    import ActionCable from 'actioncable';

    export default function cableMiddleware() {
      const cable = ActionCable.createConsumer('/cable');

      return ({ dispatch, getState }) => next => (action) => {
        if (typeof(action) === 'function') {
          return next(action)
        }

        const {
          channel,
          room,
          leave,
        } = action;
        let { received } = action;

        if (!channel) {
          return next(action);
        }

        if (leave) {
          const subscription = _.find(
            cable.subscriptions.subscriptions,
            sub => sub.identifier === JSON.stringify({ channel, room }),
          );

          return cable.subscriptions.remove(subscription);
        }

        if (typeof(received) === 'string') {
          received = result => dispatch({ type: received, result })
        }

        return cable.subscriptions.create({ channel, room }, { received });
      };
    }

Lets break down this code:

Basically we first skip our middleware if the action is a function or if there is no `channel` attribute in our action.

Then if there is a `leave` attribute, then we remove the action cable subscription to the channel+room.

Else we create a subscription to the channel+room.

But you will notice that we are doing some quick logic to check if the `received` attribute is a string. And if it is, we are changing its value to a function that dispatches a new action with the received data. So basically our `received` attribute can take both an action type string or an actual function. This gives us an extra level of control over how we handle the data coming in from Action Cable.

And any data that is sent from the ActionCable server is included in the `result` attribute of the action. So the reducer can have easy access to any data the server sent. Also any extra action attributes (`...rest`) are just passed directly through to the dispatched action.

Also notice that the function that we are exporting is returning another function. This is an example of a [Higher-order function](https://en.wikipedia.org/wiki/Higher-order_function "https://en.wikipedia.org/wiki/Higher-order_function") and is a very useful pattern in Javascript. We will execute the outside function when we apply the middleware to redux which will create the ActionCable connection only once and give the inner function access to that cable connection going forward.

### Add Middleware to Redux

Then we have to apply our new middlware. Check the [redux documentation](https://redux.js.org/advanced/middleware#attempt-6-naively-applying-the-middleware "https://redux.js.org/advanced/middleware#attempt-6-naively-applying-the-middleware") for how to do this. But you will probably need to do something like this when setting up your store:

    import { createStore, applyMiddleware } from 'redux';
    import cableMiddleware from './middleware/cableMiddleware';
    import rootReducer from './reducers/index';

    const store = createStore(
      rootReducer,
      applyMiddleware(cableMiddleware())
    );

Since other file is actually exporting a higher-order function, don't forget to execute the cableMiddleware function when applying it to redux. This will also create the Action Cable connection only once when the store loads so we don't have to worry about creating a new connection every time.

### Our New Action Creators

We now have access to a new type of action that has new required attributes. Basically if we dispatch an action with a `channel` attribute it will trigger our cable middleware.

Here are some example action creators using our new middleware.

    export function subscribeConversation(conversationId) {
      return {
        channel: 'conversations',
        room: `conversation_${conversationId}`,
        received: NEW_MESSAGE,
      }
    }

    export function unsubscribeConversation(conversationId) {
      return {
        channel: 'conversations',
        room: `conversation_${conversationId}`,
        leave: true,
      }
    }

    // Action creator with received function:
    export function subscribeConversation(conversationId) {
      return dispatch => dispatch({
        channel: 'conversations',
        room: `conversation_${conversationId}`,
        received: data => dispatch({
          type: NEW_MESSAGE,
          payload: data.conversation,
        }),
      });
    }

You'll notice these actions don't even have the required `type` attribute, this is because when they are dispatched we hijack the action and do out own thing, so these particular actions never makes it to the reducer.

Instead we have a `channel`, `room`, and `received` attributes required to subscribe to a channel+room, and `channel`, `room`, and `leave`, attributes required to unsubscribe from a channel+room.

The important part here is that `received` is either an action string to dispatch when new data comes in, or a function to run when new data comes in.

### Conclusion

I'm now a huge fan of redux middleware, I love how it cleans up and simplifies doing complex repetative things in our action creators.

Now we've set this up we can subscribe and unsubscribe to rooms and channels very easily, as well as handle the data from the server in very robust and dynamic ways.
