---
title: Adding ESLint to Your Project with a Git Hook
layout: post
date: 2018-07-17 00:00:00 +0000
hero: "/uploads/2018/07/17/Beautiful Code!.png"
tags:
- javascript
- git

---
## Installation

First install the package eslint:

    yarn add eslint --only=dev

Then initialize init and it will walk you through a typical setup:

    yarn eslint --init

These are the options I selected. I like to start with the AirBnB template:

    ? How would you like to configure ESLint? Use a popular style guide
    ? Which style guide do you want to follow? Airbnb
    ? Do you use React? Yes
    ? What format do you want your config file to be in? JSON

The airbnb template is already set up to use react and it will install all sorts of nice dependencies for you.

And thats basically it!

## Configuration

It creates a file called `eslintrc.json` (the file will be different depending on which format you chose). And in this file will be your configuration.

If you open that file it will look like this:

    {
      "extends": "airbnb",
    }

So it is basically just copying all of airbnb's rules. So if we want to now change the rules we are using we need to manually add them. We can turn off or change any rule and any rule we specify in our file will override the airbnb rule.

Here is an example of how to disable a rule called `no-underscore-dangle` and change the rule called `react/jsx-filename-extension` which requires all javascript files what use jsx to have the extension `.jsx`. This rule change also allows you to have `.js` extension.

    {
      "extends": "airbnb",
      "rules": {
        "react/jsx-filename-extension": [1, { "extensions": [".js", ".jsx"] }],
        "no-underscore-dangle": 0
      }
    }

To disable eslint totally on a specific line, you can add this at the end of the line:

    var foo = "bar" // eslint-disable-line

And to disable 1 rule on that line you can add the rule like this:

    var foo = "bar" // eslint-disable-line no-alert

You can also do it this way:

    // eslint-disable-next-line
    var foo = "bar"
    
    // eslint-disable-next-line no-alert
    var foo = "bar"

You can lint any file by running `yarn eslint path/to/file.js`.

You can also configure your editor to display any lint issues inline for you.

You can also set up a git pre-commit script to run before each commit which will enforce the lint rules in a very strong way.

## Setting up a ESLint pre-commit git hook

I'm assuming you already have a git repo setup.

Create a new file:

    touch .git/hooks/pre-commit

Make the file executable:

    chmod +x .git/hooks/pre-commit

Then paste in this code here into that file:

    #!/bin/sh
    
    STAGED_FILES=$(git diff --cached --name-only --diff-filter=ACM | grep ".jsx\{0,1\}$")
    ESLINT="$(git rev-parse --show-toplevel)/node_modules/.bin/eslint"
    
    if [[ "$STAGED_FILES" = "" ]]; then
      exit 0
    fi
    
    PASS=true
    
    printf "\nValidating Javascript:\n"
    
    # Check for eslint
    if [[ ! -x "$ESLINT" ]]; then
      printf "\t\033[41mPlease install ESlint\033[0m (npm i --save-dev eslint)"
      exit 1
    fi
    
    for FILE in $STAGED_FILES
    do
      "$ESLINT" "$FILE"
    
      if [[ "$?" == 0 ]]; then
        printf "\t\033[32mESLint Passed: $FILE\033[0m"
      else
        printf "\t\033[41mESLint Failed: $FILE\033[0m"
        PASS=false
      fi
    done
    
    printf "\nJavascript validation completed!\n"
    
    if ! $PASS; then
      printf "\033[41mCOMMIT FAILED:\033[0m Your commit contains files that should pass ESLint but do not. Please fix the ESLint errors and try again.\n"
      exit 1
    else
      printf "\033[42mCOMMIT SUCCEEDED\033[0m\n"
    fi
    
    exit $?

This code is from \[this article\]([https://medium.com/@shettyrahul8june/how-to-run-eslint-using-pre-commit-hook-25984fbce17e](https://medium.com/@shettyrahul8june/how-to-run-eslint-using-pre-commit-hook-25984fbce17e "https://medium.com/@shettyrahul8june/how-to-run-eslint-using-pre-commit-hook-25984fbce17e")) which references \[this gist\]([https://gist.github.com/rashtay/328da46a99a9d7c746636df1cf769675](https://gist.github.com/rashtay/328da46a99a9d7c746636df1cf769675 "https://gist.github.com/rashtay/328da46a99a9d7c746636df1cf769675")).

Now anytime someone commits, any javascript files included in the commit will have to pass the lint test!