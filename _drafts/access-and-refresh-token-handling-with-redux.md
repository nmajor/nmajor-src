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

Now this step is going to vary a lot depending on your implementation. Personally I was greatly inspired by [this library](https://github.com/erikras/react-redux-universal-hot-example) which uses middleware that looks like [this](https://github.com/erikras/react-redux-universal-hot-example/blob/master/src/redux/middleware/clientMiddleware.js), so you can dispatch API requests that look like redux actions. Really cool.

Regardless, I'm going to create another middleware and load it into redux ahead of my API middleware. This new piece of middleware will check if the dispatched action is an API request, and if it is, it will check if the accessToken is expired. If it's not, it will `next()` and if it is it will refresh the token, dispatch a redux action to store the new set of tokens inside `state.auth.tokens` and then `next()` so the request has the new accessToken to use.

The middleware is in a file called `tokenMiddleware.js` and looks like this:

```

```

You'll notice I already loaded the tokens with a `expires_at` attribute. I had to calculate this myself when I got the response back from the auth server.


