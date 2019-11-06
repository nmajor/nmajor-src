---
layout: post
title: Document with JSDoc + Swagger
hero: ''
tags: []
date: 

---
First install JSDoc in your project:

    yarn add --dev jsdoc

Then add a config file for JSDoc:

    touch ./jsdoc.json

    {
        "source": {
            "includePattern": ".+\\.js(doc|x)?$",   // Only process file ending in .js, .jsdoc or .jsx
            "include": ["."],                       // Check all folders.
            "exclude": ["node_modules"]             // Be gone, node_modules.
        },
        "recurseDepth": 10,                         // Only go 10 levels deep.
        "opts": {
            "destination": "./docs/",               // Where I want my docs to be generated.
            "recurse": true                         // Same as using -r or --recurse
        }
    }

Then create add an npm script that runs jsdoc with our new config file:

    "scripts": {
        "docs": "jsdoc -c ./jsdoc.json"
    }

Then you can generate the docs by running the npm script:

    yarn docs