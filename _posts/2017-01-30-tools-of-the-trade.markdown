---
layout: post
title:  "DevOps tools of the trade"
date:   2017-01-25 11:14:49 +0200
categories: devops learn tips
---

## Introduction

A fair amount of time while "devopsing" is spent on troubleshooting.

On many occations it is a battle to figure out why the deployment process, that in itself is a combination of integrations between various softwares, is misplacing, or misconfiguring files on the system you are attempting to integrate/deploy.

On a recent project I was asked to remove unwanted files from the git repo. A seemingly strightforward task, instead of having the files in the git repo, they should be added during the deployment process, in this case copied from AWS S3 to the EC2 server.

So I proceeded to `git rm` the files, created a folder in an [S3 Bucket](http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingBucket.html), created an [IAM policy](https://aws.amazon.com/blogs/security/writing-iam-policies-how-to-grant-access-to-an-amazon-s3-bucket/) added it to the EC2 instance [role](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)

I then created an [AWS ElasticBeanstalk extention config file](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/ebextensions.html) that pulls the files from s3 to the right place on the server. 

Something like this `.ebextensions/privatefiles.config`:

```
files:
  /var/tmp/s3pullprivatefiles.sh:
    mode: "000755"
    owner: root
    group: root
    content: |
      #!/usr/bin/env bash

      #download the files from s3
      aws s3 cp s3://account.privatebucket/private-file1.ext /var/app/current/resources
      aws s3 cp s3://account.privatebucket/private-file2.ext /var/app/current/resources

container_commands:
  99private-files:
    command: "/var/tmp/s3pullprivatefiles.sh"
```

Everything seemed to be in place

So I commited the code to git, and pushed to the server, and the Continuous Integration process kicked in. The build started in [Bamboo](https://www.atlassian.com/software/bamboo), once completed I deployed the Release with the Artifact just created. 

The deployment process included a Task to deploy to AWS ElasticBeanstalk, and even though I had an example from another `.ebextention` config file that pulls the SSL certificates from S3 to the right place - that files I was looking to pull did not appear under `/var/app/current/resources`.

And so the troubleshooting begins 

## Searching in logs

Before you can search in log files, you need to find them ...

Seems trivial, but it's not always the case. While the Linux convention is that logs can be found under `/var/log/`, in this case, the logs needed to be accessed from AWS ElasticBeanstalk, through the Console

![elasticbeanstalk-logs]({{ site.url }}/assets/environment-management-logs.png){:class="img-responsive"}

**TO BE CONTINUTED ...**

## Quick-scripting to check state of files in the system


