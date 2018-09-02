---
title: Serverless Framework Executables with Webpack - AWS Lambda
layout: post
date: 2018-09-01 00:00:00 +0000
hero: "/uploads/2018/09/02/Serverless Framework with Webpack-Executables.png"
tags:
- serverless
- javascript
- aws
- webpack

---
Many web apps rely on executable binaries to function. For example if you want to do any kind of image processing, usually, in addition to the actual libraries you are using usually you need an program like \`imagemagick \` to make it actually work.

So, if you ever want to build a sophisticated web app with the serverless framework, you need to be able to upload and use executable binaries. And its best if you can upload them in such a way that the library knows how to find them without any extra configuration.

{: .lead}  
<!–-break-–>

### The Approach

Basically we need to:

* Upload the executable binaries with the aws lambda deployment package.
* Make sure our executable binaries maintain the correct executable permissions.
* Change the `$PATH` env variable to include the location of our executables.

And then we should be good to go.

### Upload

##### Without Webpack

If you're not using the `serverless-webpack` plugin, then this is actually pretty easy.

I like to just create a folder in my project called `bin` and put them all in there. I did this in my [Serverless Backend for React](http://nmajor.com/posts/serverless-back-end-for-react-your-introduction-to-serverless-architecture "http://nmajor.com/posts/serverless-back-end-for-react-your-introduction-to-serverless-architecture") post.

After you put the executables in the `bin` folder, put this in your `serverless.yml` file:

    package:
      include:
        - bin/*

Your files should maintain their permissions.

##### With Webpack:

Here is where it gets tricky. The package includes are not honored by the `serverless-webpack` plugin, so we have have webpack include the files in our bundle. We're going to use the official [copy-webpack-plugin](https://webpack.js.org/plugins/copy-webpack-plugin/ "https://webpack.js.org/plugins/copy-webpack-plugin/") to do this, but here's the kicker. The [copy-webpack-plugin](https://webpack.js.org/plugins/copy-webpack-plugin/ "https://webpack.js.org/plugins/copy-webpack-plugin/") does NOT preserve file permission, so we then have to write a custom plugin to set the file permissions of all the files in our `bin` folder to `755` so they can be executed by lambda.

But first lets copy our files with [copy-webpack-plugin](https://webpack.js.org/plugins/copy-webpack-plugin/ "https://webpack.js.org/plugins/copy-webpack-plugin/"). Do do this you can checkout the [documentation here](), or you can add this to your webpack config:

      plugins: [
        new CopyWebpackPlugin([{
          from: 'bin/',
          to: 'bin/'
        }]),
      ]

Pretty easy.

### Permissions

I did find a plugin called [webpack-permissions-plugin ](https://www.npmjs.com/package/webpack-permissions-plugin "https://www.npmjs.com/package/webpack-permissions-plugin") that helped me solve the issue with the permissions, but I couldn't get the feature to change all the permissions in the directory to work. So I decided to make my own simplified version of that plugin to simply handle changing the permissions of the files in the bin folder.

So here's the simple plugin I whipped up:

    const fs = require('fs');
    
    function WebpackBinPermission(options) {
      this.options = options || {};
    }
    
    WebpackBinPermission.prototype.apply = function (compiler) {
      compiler.plugin('done', () => {
        const permissions = this.options.permissions || '755';
        const binPath = `${compiler.outputPath}/bin`;
        fs.readdir(binPath, (err, items) => {
          if (items && items.length > 0) {
            for (let i = 0; i < items.length; i += 1) {
              fs.chmodSync(`${binPath}/${items[i]}`, permissions);
            }
          }
        });
      });
    };
    
    module.exports = WebpackBinPermission;

As you can see, its pretty simple and all it does it changes the permissions of everything in the `/bin` folder to `755`. And for me it works like a charm.

To then to use it, you just have to pair it with the [copy-webpack-plugin](https://webpack.js.org/plugins/copy-webpack-plugin/ "https://webpack.js.org/plugins/copy-webpack-plugin/") in your webpack config like this:

      plugins: [
        new CopyWebpackPlugin([{
          from: 'bin/',
          to: 'bin/'
        }]),
        new WebpackBinPermission(),
      ]

And then when [serverless-webpack]() builds it will automatically copy everything in the bin folder and set the executable permissions correctly.

### Executable Path

According to the [aws lambda documentation](https://docs.aws.amazon.com/lambda/latest/dg/current-supported-versions.html "https://docs.aws.amazon.com/lambda/latest/dg/current-supported-versions.html"), lambda functions run in a linux environment. And libraries are used to having access to ready to run executables. That means that in a terminal you can just type in the name of the program without referencing the whole path. If you're not familiar with how this works take a look at [this page by linfo.org](http://www.linfo.org/path_env_var.html "http://www.linfo.org/path_env_var.html").

To make an executable ready-to-run without referencing its path, we have to set the `$PATH` environmental variable and tell linux to look inside our own `bin` folder when searching for valid ready-to-run executables.

Thats actually really easy. According to [this official aws post]() we can do it by just putting this line at the top of our handler code:

    process.env.PATH = `${process.env.PATH}:${process.env.LAMBDA_TASK_ROOT}/bin`;

But I like to also wrap it around some logic so it doesnt interfere with testing:

    if (process.env.LAMBDA_TASK_ROOT) {
      process.env.PATH = `${process.env.PATH}:${process.env.LAMBDA_TASK_ROOT}/bin`;
    }

### Wrap it up

That is the approach that worked for me. I'm currently doing a lot of stuff with executables, and so I needed a solution that allowed me to possibly put many many files in the bin folder that reference each other, and this approach seems to work well.

As always, please comment below if you see any room for improvement in my code, approach, or writing.

Thanks for reading!