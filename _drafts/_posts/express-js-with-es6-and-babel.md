---
title: Express.js With ES6 and Babel
layout: post
date: 2018-02-16 00:00:00 +0000
tags:
- express
- node
- babel
- es6
hero: "/uploads/2018/02/16/Express.js - ES6 - Babel.png"
---
Express.js doesn't come with es6 out of the box. But its actually not that hard to add.

This is the best way I have found to add ES6 support to an express project. I'm going to go over how to easily add ES6 support to your project using the [babel-cli](https://www.npmjs.com/package/babel-cli "babel-cli").

The `babel-cli` gives us a command line tool called `babel-node` that wraps around node and converts our ES6 code to basic javascript that node can read.

**However it is not recommended to use babel-node in production**

### New Express App

First lets use the [express-generator](http://expressjs.com/en/starter/generator.html) to make a new Express.js app.

    $ express --view pug --css sass new-app
    $ cd new-app
    $ npm install

This generates a new express app in a new folder called new-app. And as you can see, I've told express to use sass for css and pug for the html templating.

### Rearrange Files

We are going to need to rearrange things a bit. This is because of the production build. We want all the javascript files isolated in a subfolder of our app because later we are going to tell babel to compile all the files and things get really messy if all our code is in at the same level as our dot files, node_modules folder, etc...

So lets make a new folder called `app` and move all our express stuff into it.

    $ mkdir app
    $ mv bin public routes views app.js app/

### Install Babel

First we need to add 2 babel packages to our apps dev-dependencies.

    $ npm install babel-cli --save-dev
    $ npm install babel-preset-env --save-dev

[babel-preset-env]() by default behaves like [babel-preset-latest](). Basically it includes all the new features of ECMAScript that are standardized each year. But it also allows you to specify specific environments to build for such as specific browsers you want to support.

Next we need to add the babel configuration file `.babelrc` to our app folder. Make it look like this:

    {
      "presets": ["env"]
    }

This just tells babel to use [babel-preset-env]() that we just installed but take a look as some of the other configuration settings you can specify [here](https://babeljs.io/docs/usage/babelrc/). 

### Try it Out

To try it out lets open our `app.js` file and replace this line

    var express = require('express');

with this:

    import express from 'express';

And then we run express with babel with this command:

    $ babel-node app/bin/www

It works!

And you will notice that if you just run `node app/bin/www` you get an error which is expected.  
  
Now lets add a npm script to make running this easier. Open up your `package.json` file and lets add a new line in the scripts section so it looks like this:

    ...
      "scripts": {
        "start": "node ./bin/www",
        "start:dev": "babel-node ./app/bin/www"
      }
    ...

Now we can run our babelized app with the command:

    npm run start:dev

You might be asking, "Why don't we just use the existing start script". We need to save that for production because many production environments use the start script to run the app, but obviously we need to change it since we know `node ./bin/www` doesn't work.

### Preparing for Production

So to prepare our app for production we need a build step that will take our ES6 code and generate node-friendly ES5 code in a file somewhere that node can run.

To do this I like to another npm script:

    ...
      "scripts": {
        "start": "node ./bin/www",
        "start:dev": "babel-node ./app/bin/www",
        "build": "rm -rf ./build && mkdir build && babel --out-dir ./build --source-maps --copy-files ./app"
      }
    ...

As you can see this build script removes and recreates any existing `build` folder, and then runs the command `babel --out-dir ./build --source-maps --copy-files ./app` which compiles all the javascript files and copies any other files  into the `build` folder. Checkout [this page](https://babeljs.io/docs/usage/cli/) for more info on how the babel command works.

You may want to add the `build` folder to your `.gitignore`.

Then the last thing to do is to change the start script to look like this:

    "start": "node ./app/bin/www"

And we are good to go.

Just make sure you run `npm run build` as part of any deployment into production.