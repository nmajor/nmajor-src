---
title: Serverless Initial Setup With ES6, Testing, and CI Deployment
layout: post
date: 2018-08-25 00:00:00 +0000
hero: ''
tags: []

---
As a Rails developer turned Javascript Zealot, I sometimes miss the structure and opinions of the Ruby on Rails world. Its amazing how bare bones many javascript libraries are. They are so modular and self contained (good things) that, unless you take the time to add some structure and organization to your code, its easy for your project to feel chaotic an unorganized. So I'm always looking for and trying to find some good patterns and structure to follow.

The serverless framework is a good example of this. Its so minimal in its setup that it may be difficult to know where to start to give it some structure. So here I'll share with you my current initial setup I use when starting a new serverless API project.

I'm assuming you've at least tried using the serverless framework before. If you are new to serverless, I have an article that breaks it down from the beginning here:

[Serverless Back-End for React - Your Introduction to Serverless Architecture](/posts/serverless-back-end-for-react-your-introduction-to-serverless-architecture "/posts/serverless-back-end-for-react-your-introduction-to-serverless-architecture")

This includes:

* File structure
* API structure
* Testing boilerplate
* Automated deployment with Travis CI

Lets start with the great [Serverless Node.js Starter](https://serverless-stack.com/chapters/serverless-nodejs-starter.html "https://serverless-stack.com/chapters/serverless-nodejs-starter.html") ([github](https://github.com/AnomalyInnovations/serverless-nodejs-starter "https://github.com/AnomalyInnovations/serverless-nodejs-starter")) from the awesome [Serverless Stack](https://serverless-stack.com/ "https://serverless-stack.com/") project. If you're not familiar with that project be sure to check it out.

Check out [this page](https://serverless-stack.com/chapters/serverless-nodejs-starter.html "https://serverless-stack.com/chapters/serverless-nodejs-starter.html") for all the information about the starter, but it basically includes all this stuff:

* **Use ES7 syntax in your handler functions**
* **Package your functions using Webpack**
* **Run API Gateway locally**
  * Use `serverless offline start`
* **Support for unit tests**
  * Run `npm test` to run your tests
* **Sourcemaps for proper error messages**
  * Error message show the correct line numbers
  * Works in production with CloudWatch
* **Automatic support for multiple handler files**
  * No need to add a new entry to your `webpack.config.js`
* **Add environment variables for your stages**

This is a great place to start. So lets get started.

### Create a New Project

First make sure you have the `serverless` module installed globally

    yarn global add serverless
    # or
    npm install serverless -g

And then run this command to create a new serverless project using the starter:

    $ serverless install --url https://github.com/AnomalyInnovations/serverless-nodejs-starter --name my-project

It gives us a `serverless.yml` file that looks like this:

    service: my-project
    
    plugins:
      - serverless-webpack
      - serverless-offline
    
    custom:
      webpack:
        webpackConfig: ./webpack.config.js
        includeModules: true
    
    provider:
      name: aws
      runtime: nodejs8.10
      stage: dev
      region: us-east-1
    
    functions:
      hello:
        handler: handler.hello
        events:
          - http:
              path: hello
              method: get
    

But, you'll see that even though the starter gives us a bunch of great stuff, it gives us no folder structure to work with

### API Folder Structure

When building an api I like to organize the paths and the folder structure

### Jest to Mocha

The the first thing I like to do is replace jest with mocha:

    mv tests test
    yarn remove jest
    yarn add mocha
    touch test/test_helper.js