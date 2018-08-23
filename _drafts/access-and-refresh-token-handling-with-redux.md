---
title: Access and Refresh Token Handling with React + Redux
layout: post
date: 2018-08-23 00:00:00 +0000
hero: ''
tags:
- react
- redux
- authentication

---
The industry trend of decoupling backends and frontends has lots of advantages. You could argue that its just good software design. Plus it makes it much easier to have multiple front-end clients using the same backend. And since mobile apps dont use cookies, then it makes sense to convert the entire authentication system to some kind of token based solution.

But the next questions is how can you safely and convienently store and manage these tokens in your React+Redux app.

{: .lead}  
<!–-break-–>

Here is the approach I've used recently and it seems to work out pretty well. Its very similar to the approach I found [here](https://michaelwashburnjr.com/best-way-to-store-tokens-redux/ "https://michaelwashburnjr.com/best-way-to-store-tokens-redux/"). In this example I'm dealing with accessToken + refreshToken, but this could easily be adapted to different kinds of tokens (a JWT auth for example).

### Break it down

First we'll make sure that redux has gets the token. In my case this means passing it from the server after the authentication callback.

Then we will use a simple redux subscriber to store our auth tokens in the browser localStorage. Using a subscriber will keep it synced so that anytime the auth token in the redux state changes it updates it in the localStorage.

Then we'll make sure that anytime we refresh the page, we load in the auth tokens from localStorage when creating the store.

Then we'll add some middleware that happens before each outgoing HTTP request that will refresh the token if needed, and update the redux state with the new tokens, and our subscriber will automatically update the token in localStorage.

### Get the Token from the Server to Redux

You can probably skip this part if you are already getting your auth tokens to redux some other way.

In my case, I'm authenticating with Microsofts Oauth2 service. and the tokens are being sent to the server via a RedirectURI callback, that means I need to pass the tokens from the server to the client. I don't store it in the session or database, Instead I pass it back to the client using the `window.__PRELOADED_STATE__` demonstrated in the [Redux server rendering documentation](https://redux.js.org/recipes/serverrendering).

I've decided to store my auth tokens in redux under `state.auth.tokens`, so my preloadedState object will look like this:

```
const preloadedState = {
  auth: {
    tokens: { /* Auth token data goes here */ }
  }
}
```

Then we set that as the value of the `window.__PRELOADED_STATE__`, in the server rendered html like this (Making sure to include these lines BEFORE loading in the client js bundle):

```
<script>
  window.__PRELOADED_STATE__ = ${JSON.stringify(preloadedState).replace(/</g, '\\u003c')}
  
  /* The string replace is to prevent injections into our preloaded state. Check the redux server rendering docs for more info */
</script>
```

Then when loading the store, load in the `window.__PRELOADED_STATE__` as the initial state when creating the redux store:

```
const store = createStore(reducer, window.__PRELOADED_STATE__)
```

### Sync Auth State to localStorage

Thanks again to [this post](https://michaelwashburnjr.com/best-way-to-store-tokens-redux/) for the idea of using a simple redux store subscriber to keep localStorage synced with out auth section of the redux store.

I made a function that will serialize and set the localStorage variables, and then call that function from `store.subscribe`

```
function setAuthState(state) {
  try {
    localStorage.setItem('state.auth.tokens', JSON.stringify((state.auth || {}).tokens));
  } catch (err) { return undefined; }
}

store.subscribe(() => {
  setAuthState(store.getState())
});
```

Be sure to checkout the [documentation for store.subscribe](https://redux.js.org/api/store#subscribe). You can also checkout [this video](https://egghead.io/lessons/javascript-redux-persisting-the-state-to-the-local-storage) by Dan Abramov on how to use subscribe to store the state into localStorage. 

Some other libraries for watching the state for mutations are [redux-watch](https://github.com/jprichardson/redux-watch) and [redux-subscribe](https://github.com/ashaffer/redux-subscribe). They may be a better option later, but for now I'm going with this simple subscribe approach.

### Load Auth From localStorage on Refresh

Then I made a function to get and deserialize the state from localStorage:

```
function getAuthState() {
  try {
    const tokens = JSON.parse(localStorage.getItem('state.auth.tokens')) || undefined;
    const user = JSON.parse(localStorage.getItem('state.auth.user')) || undefined;

    return { auth: { tokens, user } }
  } catch (err) { return undefined; }
}
```

Then change our create store to something like this:

```
const store = createStore(
  reducer,
  { ...getAuthState(), ...window.__PRELOADED_STATE__ }
 )
```

And I was able to refresh the page and the auth tokens persist. So it works!

### Automatic Token Refreshing

I debated whether or not to include this part 

Now this step is going to vary a lot depending on your implementation. I was greatly inspired by [this library (erikras/react-redux-universal-hot-example)](https://github.com/erikras/react-redux-universal-hot-example) which has a really nice middleware setup for making API requests by dispatching redux actions.

Although my final implementation looks much more like that the middleware found in the [erikras/react-redux-universal-hot-example](https://github.com/erikras/react-redux-universal-hot-example) mentioned above, here is an exmample of how you might do a simplified version of some API request middleware with the refresh token.

I'm assuming you are using a function dispatcher middleware like [redux-thunk](https://github.com/reduxjs/redux-thunk) and [superagent](https://github.com/visionmedia/superagent) as the request library.

You'll also notice that I loaded my tokens with an `expires_at` attribute to help calculate when a refresh is needed.

The middleware is in a file called `requestMiddleware.js` and looks something like this like this:

```
export default function requestMiddleware() {
  return ({ dispatch, getState }) => next => (action) => {
    const {
      request,
    } = action;

    if (!request) {
      return next(action);
    }

    const { tokens } = getState().auth;

    // 5 minutes from now
    const refreshThreshold = (new Date.getTime() + 300000);

    if (tokens.refresh_token && refreshThreshold > tokens.expires_at) {
      return superagent.post('/path/to/renew')
        .send({ refresh_token: tokens.refresh_token })
        .end((err, { body } = {}) => {
          dispatch({ type: 'SET_TOKENS', payload: body });
          request(body);
        });
    }
    return request(tokens);
  };
}
```

Dont forget to apply the middleware:

```
import { createStore, applyMiddleware } from 'redux';
import requestMiddleware from './middleware/requestMiddleware';
import rootReducer from './reducers/index';

const store = createStore(
  rootReducer,
  applyMiddleware(requestMiddleware())
);
```

Add a reducer to capture the `SET_TOKEN` action:

```
export default function reducer(state = initialState, action = {}) {
  switch (action.type) {
    case SET_TOKENS:
      return {
        ...state,
        auth: { tokens: action.payload },
      };
    default:
      return state;
  }
}
```

And now we can dispatch actions like this:

```
export default function sendAnyRequest() {
  return (dispatch) => {
    return {
      request: (tokens) => superagent.post('/some/random/request')
        .send({ foo: 'bar' })
        .end((err, { body } = {}) => {
          dispatch({ type: 'SOME_EVENT', payload: body });
        });
    }
  }
}
```

### Wrap it up

And before making any request it will first check the validity of the refresh token and refresh it if needed. And if a refresh does occur it will set the new token in the redux store, which will be automatically written to the localStorage by the subscriber.

Noice!

I highly recommend you read through the code of this project [erikras/react-redux-universal-hot-example](https://github.com/erikras/react-redux-universal-hot-example), specifically the `ApiClient.js`, `clientMiddleware.js`, and see how the request actions are dispatched.

As I said, I used that example heavily when setting up my redux store. Here is what my middleware file looks like:

```
import { SIGN_OUT, SET_TOKENS } from '../modules/auth';
const refreshThreshold = (new Date().getTime() + 300000); // 5 minutes from now

export default function clientMiddleware(client) {
  return ({ dispatch, getState }) => next => (action) => {
    if (typeof action === 'function') {
      return action(dispatch, getState);
    }

    const {
      promise, types, ...rest
    } = action;
    if (!promise) {
      return next(action);
    }

    // eslint-disable-next-line no-param-reassign
    client.token = (getState().auth.tokens || {}).access_token;

    const [REQUEST, SUCCESS, FAILURE] = types;
    next({ ...rest, type: REQUEST });

    let actionPromise = Promise.resolve();
    const { tokens } = getState().auth;

    if (tokens.refresh_token && refreshThreshold > tokens.expires_at) {
      actionPromise = client.post('/my-server/renew', { data: { refresh_token: tokens.refresh_token } })
        .then(
          (result) => {
            client.token = result.access_token;
            return next({
              ...rest, result, type: SET_TOKENS,
            });
          },
          errors => next({
            ...rest, errors, type: SIGN_OUT,
          }),
        )
        .then(() => promise(client));
    } else {
      actionPromise = promise(client);
    }

    actionPromise.then(
      result => next({ ...rest, result, type: SUCCESS }),
      errors => next({ ...rest, errors, type: FAILURE }),
    ).catch((error) => {
      console.error('MIDDLEWARE ERROR:', error);
      next({ ...rest, error, type: FAILURE });
    });

    return actionPromise;
  };
}

```

And I can dispatch really clean actions that look like this:

```
export function loadOne(_id) {
  return {
    types: [LOAD, LOAD_SUCCESS, LOAD_FAIL],
    promise: client => client.get('/items/'),
  };
}
```

And the tokens are automatically refreshed and persisted.
