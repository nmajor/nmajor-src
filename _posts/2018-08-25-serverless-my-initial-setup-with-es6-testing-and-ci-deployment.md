---
title: Robust Serverless API Boilerplate with ES6, Folder Structure, Testing (Mocha
  + Chai), and ESLint
layout: post
date: 2018-08-25 00:00:00 +0000
hero: "/uploads/2018/08/25/robust.png"
tags:
- serverless
- eslint
- javascript
- es6

---
As a Rails developer turned Javascript Zealot, I sometimes miss the structure and opinions of the Ruby on Rails world. Its amazing how bare bones many javascript libraries are. They are so modular and self contained (good things) that, unless you take the time to add some structure and organization to your code, its easy for your project to feel chaotic an unorganized. So I'm always looking for and trying to find good patterns and structure to follow with my javascript projects.

The serverless framework is a good example of this. Its so minimal in its setup that it may be difficult to know where to start to give it some structure. So here I'll share with you one possible way to structure a serverless API project.

{: .lead}  
<!–-break-–>

Basically before I start any serious project I like to have a few things setup:

* ES6/ES7 Webpack and Babel (From Starter)
* A good offline dev workflow (From Starter)
* ESLint
* Folder structure
* Testing

I hope this isnt too obscure of a topic, but I'm not actually going to cover how to build a CRUD API, this is just an example starting boilerplate before you begin coding your project.

I'm assuming you've at least tried using the serverless framework before. If you are new to serverless, I have an article that breaks it down from the beginning here:

[Serverless Back-End for React - Your Introduction to Serverless Architecture](/posts/serverless-back-end-for-react-your-introduction-to-serverless-architecture "/posts/serverless-back-end-for-react-your-introduction-to-serverless-architecture")

### The Starter

Lets start with the great [Serverless Node.js Starter](https://serverless-stack.com/chapters/serverless-nodejs-starter.html "https://serverless-stack.com/chapters/serverless-nodejs-starter.html") ([github](https://github.com/AnomalyInnovations/serverless-nodejs-starter "https://github.com/AnomalyInnovations/serverless-nodejs-starter")) from the awesome [Serverless Stack](https://serverless-stack.com/ "https://serverless-stack.com/") project. If you're not familiar with that project be sure to check it out.

The starter basically includes all this stuff (From the the [starter description page](https://serverless-stack.com/chapters/serverless-nodejs-starter.html "https://serverless-stack.com/chapters/serverless-nodejs-starter.html")):

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

If you dont use this starter, you have to add a lot of this stuff one by one by including and configuring the right packages and plugins. So this gives us a great place to start.

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

And a handler.js file that looks like this:

    export const hello = async (event, context, callback) => {
      const response = {
        statusCode: 200,
        body: JSON.stringify({
          message: `Go Serverless v1.0! ${(await message({ time: 1, copy: 'Your function executed successfully!'}))}`,
        }),
      };
    
      callback(null, response);
    };
    
    const message = ({ time, ...rest }) => new Promise((resolve, reject) => 
      setTimeout(() => {
        resolve(`${rest.copy} (with a delay)`);
      }, time * 1000)
    );

It also gives us a test folder with an example test.

    # tests/handler.test.js
    
    import * as handler from '../handler';
    
    test('hello', async () => {
      const event = 'event';
      const context = 'context';
      const callback = (error, response) => {
        expect(response.statusCode).toEqual(200);
        expect(typeof response.body).toBe("string");
      };
    
      await handler.hello(event, context, callback);
    });

We can run this command to start the offline server for a good development workflow:

    serverless offline start

### Add ESLint

I always work with a linter these days, there's no better way to keep clean code and enforce best practices with a language like javascript where its so easy to write messy and ugly code.

Lets add `eslint` and some plugins:

    , yarn add --dev eslint eslint-config-airbnb eslint-plugin-import eslint-plugin-mocha eslint-plugin-promise

And then make a new `.eslintrc.json` file in the root of our project.

    touch .eslintrc.json

And add this to the new file:

    {
      "extends": ["airbnb/base", "plugin:promise/recommended"],
      "plugins": ["promise"],
      "rules": {}
    }

Add a `.eslintignore` file:.

    touch .eslintignore

And ignore the webpack config:

    # .eslintignore
    
    webpack.config.js

And then add a `lint` script to our `package.json`:

      "scripts": {
        "lint": "node_modules/.bin/eslint ."
      },

Then you can run the linter with this command:

    yarn lint
    # or 
    npm run lint

### API Folder Structure

When I'm building a serverless api I like to give my functions, paths, and folder a api-like structure. So I'll create some directories to organize my handlers. This may be a bit overkill for small projects, but I dont think there are any downsides of having this kind of extra organization right off the bat.

    mkdir -p handlers/api/v1/todos

I think you can think of these handlers as controllers, and so if you're building CRUD endpoints for a model, you can add a different file for each of the 5 main API actions:

    touch handlers/api/v1/todos/index.js
    touch handlers/api/v1/todos/show.js
    touch handlers/api/v1/todos/create.js
    touch handlers/api/v1/todos/update.js
    touch handlers/api/v1/todos/delete.js

Then the functions part of your `serverless.yml` file would look something like this:

    functions:
      api/v1/todos/index:
        handler: handlers/api/v1/todos/index.default
        events:
          - http:
              path: api/v1/todos
              method: get
      api/v1/todos/show:
        handler: handlers/api/v1/todos/show.default
        events:
          - http:
              path: api/v1/todos/{id}
              method: get
      api/v1/todos/create:
        handler: handlers/api/v1/todos/create.default
        events:
          - http:
              path: api/v1/todos
              method: post
      api/v1/todos/update:
        handler: handlers/api/v1/todos/update.default
        events:
          - http:
              path: api/v1/todos/{id}
              method: put
      api/v1/todos/delete:
        handler: handlers/api/v1/todos/delete.default
        events:
          - http:
              path: api/v1/todos/{id}
              method: delete

As you can see this is a pretty typical REST setup.

We can now remove our original handler.js file since we dont need it anymore:

    rm handler.js

We wont be using this today, but lets also create a `models` folder where I can put our models:

    mkdir models
    touch models/todo.js

Then the individual handlers will include the models to handle the crud operations.

Now I'm going to add a basic handler for `todos/index.js` so we have something to test:

    # handlers/api/v1/index.js
    
    export default (event, context, callback) => {
      const response = {
        statusCode: 200,
        body: JSON.stringify({
          message: 'Hello from todos/index',
        }),
      };
    
      callback(null, response);
    };

And if you are following along, you'll have to add an export to all the handler files in order to run the offline server.

### Setup Testing

By default the starter comes with `jest` but I prefer `mocha` so lets swap `jest` for `mocha` and add a bit of `chai` for the assertions.

    mv tests test
    rm test/handler.test.js
    yarn remove --dev jest
    yarn add --dev mocha chai
    touch test/test_helper.js

Now lets add some test files for our endpoints:

    mkdir -p test/api/v1/todos/
    touch test/api/v1/todos/index.test.js
    touch test/api/v1/todos/show.test.js
    touch test/api/v1/todos/create.test.js
    touch test/api/v1/todos/update.test.js
    touch test/api/v1/todos/delete.test.js

And we'll only add 1 test for now for our `todos/index` handler:

    import { expect } from 'chai';
    
    import todosIndex from '../../../../handlers/api/v1/todos';
    
    describe('Fetching list of todos', () => {
      it('returns a valid response', (done) => {
        const event = 'event';
        const context = 'context';
        const callback = (error, response) => {
          expect(response.statusCode).to.equal(200);
          expect(typeof response.body).to.equal('string');
          expect(response.body).to.contain('Hello from todos/index');
          done();
        };
    
        todosIndex(event, context, callback);
      });
    });

Then we can run it by first adding this script to our `package.json` file:

      "scripts": {
        "lint": "node_modules/.bin/eslint .",
        "test": "NODE_ENV=test node_modules/.bin/mocha --recursive --require babel-core/register"
      },

We have to add the `--recursive` tag so it will find the tests in our subfolders, and the `--require babel-core/register` tag to make ES7 work with our tests.

Then we can run our tests with the command:

    yarn test
    # or
    npm test

Now we also need a different `.eslintrc.json` file for our testing so if we put a new one in our test tile the tests will play by different eslint rules.

    touch test/.eslintrc.json

With these contents so it will ignore mocha keywords:

    {
      "extends": ["airbnb/base", "plugin:promise/recommended"],
      "plugins": ["promise", "mocha"],
      "env": {
        "mocha": true
      },
      "rules": {}
    }

### Wrap it up

Now we can start our offline server:

    serverless offline start

We can test our code:

    yarn test
    # or
    npm test

...and we can lint our code:

    yarn lint
    # or
    npm run lint

And we have some good folder structure to start building our API.

Thanks for reading, I hope this can be of help to someone.

As always if you notice anything I did that could be improved, please reach out. I'm always looking to improve.