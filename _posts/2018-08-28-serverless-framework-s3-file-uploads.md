---
title: Serverless Framework S3 Permissions (Serverless IAM Permissions)
layout: post
date: 2018-08-28 00:00:00 +0000
hero: ''
tags:
- serverless
- javascript
- aws

---
This really isn't a complicated problem, but I want to document this for later.

Its hard to find a good title for this. Usually you will never us a lambda function to upload to S3. For user submitted files, the right way to upload to S3 is generate a temporary signed upload URL and the user will submit directly to S3 without sending the file to the serverless function.

But I do a lot of PDF generation with one of my projects so I need to be able to put S3 objects from the serverless function, so I need a bit of setup to do that.

I created a new bucket in S3.

Then the main thing is find the simplest way to grant the serverless project access to the S3 bucket.

According to [this post](https://serverless.com/framework/docs/providers/aws/guide/iam/ "https://serverless.com/framework/docs/providers/aws/guide/iam/") the serverless framework automatically creates a IAM user for the serverless project. There are ways to customize this, but I'm going to just grant that default IAM user access to S3 and then I should be good to go.

Luckily thats as easy as adding this to the `serverless.yml` file:

    provider:
      ...
      iamRoleStatements:
        - Effect: "Allow"
          Action:
           - "s3:PutObject"
           - "s3:DeleteObject"
          Resource: { "Fn::Join": ["", ["arn:aws:s3:::BUCKET_NAME", "/*" ] ] }

[This stackoverflow question](https://stackoverflow.com/questions/46098173/how-to-add-iamrolestatements-to-s3-trigger-bucket-in-serverless-framework "https://stackoverflow.com/questions/46098173/how-to-add-iamrolestatements-to-s3-trigger-bucket-in-serverless-framework") was helpful in figuring this out. Also it probably would help if I understood cloudformation and aws IAM users more.