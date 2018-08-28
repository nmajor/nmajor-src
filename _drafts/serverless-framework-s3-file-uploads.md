---
title: Serverless Framework S3 File Uploads
layout: post
date: 2018-08-28 00:00:00 +0000
hero: ''
tags:
- serverless
- javascript
- aws

---
Its hard to find a good title for this. Usually you will never us a lambda function to upload to S3. For user submitted files, the right way to upload to s3 is generate a temporary signed upload URL and the user will submit directly to s3 without sending the file to the serverless function.

But I do a lot of PDF generation with one of my projects so I need to be able to put s3 objects from the serverless function, so I need a bit of setup to do that.