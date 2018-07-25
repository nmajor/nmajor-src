---
title: Using Action Cable with Redux - Websocket Redux Middleware
layout: post
date: 2018-07-25 00:00:00 +0000
hero: "/uploads/2018/07/25/Action Cable.png"
tags:
- redux
- javascript
- websockets
- action cable
- react

---
Middleware is one of the most useful and powerful features of redux. If you're unfamiliar with redux middleware, basically it is a way to insert extra behavior into dispatched redux actions.

{: .lead}  
<!–-break-–>

I'll show you how to set up middleware for both socket.io and Action Cable.

If you are unfamiliar with redux middleware, check out the \[documentation here\]([https://redux.js.org/advanced/middleware](https://redux.js.org/advanced/middleware "https://redux.js.org/advanced/middleware")). This approach is inspired by reading through the source of \[this example redux app\]([https://github.com/erikras/react-redux-universal-hot-example](https://github.com/erikras/react-redux-universal-hot-example "https://github.com/erikras/react-redux-universal-hot-example")), specifically \[this\]([https://github.com/erikras/react-redux-universal-hot-example/blob/master/src/redux/middleware/clientMiddleware.js](https://github.com/erikras/react-redux-universal-hot-example/blob/master/src/redux/middleware/clientMiddleware.js "https://github.com/erikras/react-redux-universal-hot-example/blob/master/src/redux/middleware/clientMiddleware.js")) middleware, so you may also want to check that out as well.

Basically redux action, by default, looks like this:

    { type: 'MY_ACTION_TYPE' ...some_data }

Redux actions have 1 required attribute, `type`. Anything else is just extra and is usually meant to be used by the reducer to mutate the state.

By using redux middleware we can define our own action patterns and structures. Then when any action is dispatched we can check if the action matches a specific pattern and have it do different things.

The goal was to allow an easy way to manage subscribe and unsubscribe to different channels and rooms. But part of the subscription is connecting different events to specific redux actions. Basically we want to say when `some_event` happens in Action Cable, then dispatch a certain redux action, to change the state with that new data.

First lets take a look at the action creators:

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

You'll notice these actions don't even have the required `type` attribute, this is because when they are dispatched we hijack the action and do out own thing, so this particular action never makes it to the reducer.

Instead we have a `channel`, `room`, and `received` attributes required to subscribe to a channel+room, and `channel`, `room`, and `leave`, attributes required to unsubscribe from a channel+room.

The important part here is that `received` is an action to dispatch when new data is sent to the channel+room. Then we can use our reducer to take whatever data sent into the channel+room and use that to change our state.

And, here's the middleware code:

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
          received,
          leave,
        } = action;
    
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

Basically we first skip our middleware if the action is a function or if there is no `channel` attribute in our action.

Then if there is a `leave` attribute, then we remove the action cable subscription to the channel+room.

Else we create a subscription to the channel+room.

But you will notice that we are doing some quick logic to check if the `received` attribute is a string. And if it is we are changing its value to be a function that dispatches a new action with the received data.

But this also means we can set received to be a function itself like this:

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

And that function will be triggered anytime we get data from the channel+room.

This makes our middleware quite powerful, and we can subscribe and unsubscribe to rooms and channels very easily, as well as handle the data from the server in very robust and dynamic ways.