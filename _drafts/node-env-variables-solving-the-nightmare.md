---
title: Node Env Variables - Working Around dotenv's Suckiness
layout: post
date: 2018-08-22 00:00:00 +0000
hero: ''
tags: []

---
The library that everyone uses to manage environmental variables in node is [dotenv](https://www.npmjs.com/package/dotenv "https://www.npmjs.com/package/dotenv"). I don't think I've ever had so much trouble with such a popular module.

With a background in Ruby on Rails I've been pretty annoyed with how buggy and tricky it is to load reliable environment variables into a node app. So here I'll talk about the problems I've faced and how I've solved them.

Basically what I want is to have my development environment run with the one set of environment variables and my tests run with a different set of environment variables. 

This is not a tutorial 

# Development Mode

Right off the bat I had trouble with this module. The [instructions](https://www.npmjs.com/package/dotenv "https://www.npmjs.com/package/dotenv") say to put this as early as possible in your file:

    require('dotenv').config();

Then it should make all the config files in the `.env` file available to your entire app. This sort of worked, but I found it only seemed to work some of the time and only 