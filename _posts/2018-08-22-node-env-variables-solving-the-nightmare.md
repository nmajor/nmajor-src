---
title: Node Env Variables - dotenv Workaround
layout: post
date: 2018-08-22 00:00:00 +0000
hero: ''
tags:
- node

---
The library that everyone uses to manage environmental variables in node is [dotenv](https://www.npmjs.com/package/dotenv "https://www.npmjs.com/package/dotenv"). I don't think I've ever had so much trouble with such a popular module.

What I want is to have my development environment run with the one set of environment variables and my tests run with a different set of environment variables.

{: .lead}  
<!–-break-–>

The instructions say to put this line in your code as early as possible.

    require('dotenv').config()

But that didnt work for me.

I think it might be a problem related to webpack. I could never get `dotenv` to work reliably, any env variables I referenced would be `undefined`, but then I finally found the solution in [this github issue](https://github.com/motdotla/dotenv/issues/133#issuecomment-255298822 "https://github.com/motdotla/dotenv/issues/133#issuecomment-255298822").

What you have to do is create a file called `.env.js` that looks like this:

    import dotenv from 'dotenv';
    dotenv.config({ silent: true });

And import it into your program as early as possible, like this:

    import {} from './env';

For some reason that makes it work.

# Testing

I also had trouble using env variables in my testing environment. What I found was that the `test_helper.js` file actually loads after some of my tests so I couldn't use the trick above.

What I did was create a new `env.js` file that looks like this:

    import dotenv from 'dotenv';
    dotenv.config({ path: '.env.test' });

...and load it as a setup script in the mocha command like this:

     NODE_ENV=test node_modules/.bin/mocha --recursive --require babel-core/register --require ./test/env.js --exit

And thats the only way I have been able to get dotenv to work predictably. I hope this helps someone.