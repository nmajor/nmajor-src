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

### Get the Token to the Client

In my case, I'm authenticating with Microsofts Oauth2 service. and the tokens are being sent to the server via a RedirectURI callback, that means I need to pass the tokens from the server to the client. I don't store it in the session or database, Instead I pass it back to the client using the `window.__PRELOADED_STATE__` technique mentioned in the Server Rendering Redux documentation

    window.__PRELOADED_STATE__

Which is