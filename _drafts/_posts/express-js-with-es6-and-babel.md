---
title: Express.js With ES6 and Babel
layout: post
date: 2018-02-16 00:00:00 +0000
tags:
- express
- node
- babel
- es6
hero: ''
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

### Install Babel

First we need to add 2 babel packages to our apps dev-dependencies.

    $ npm install babel-cli --save-dev
    $ npm install babel-preset-env --save-dev

babel-preset-env