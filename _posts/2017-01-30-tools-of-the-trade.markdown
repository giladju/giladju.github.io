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

The deployment process included a Task to deploy to AWS ElasticBeanstalk, and even though I had an example from another `.ebextention` config file that pulls the SSL certificates from S3 to the right place - the files I was looking to pull did not appear under `/var/app/current/resources`.

And so the troubleshooting begins 

## Searching in logs

Before you can search in log files, you need to find them ...

Seems trivial, but it's not always the case. While the Linux convention is that logs can be found under `/var/log/`, in this case, the logs needed to be accessed from AWS ElasticBeanstalk, through the Console

![elasticbeanstalk-logs]({{ site.url }}/assets/environment-management-logs.png){:class="img-responsive"}

So lets say we have a log files on our server - we know they under the present working directory - and they all have a suffix of `.log`, lets start by listing them:

```
cd /var/log    # for example
find . -name "*.log"
```

As you might imagine there are a lot of files in your directory that are NOT logs:

```
find . -not -name "*.log"
```

If we were to run a search for a string in ALL the files in the directory, like so:

```
grep -r ERROR .
```

It would take a long time and might yeild reduandant results

So let's search, only in the log files:

```
find . -name "*.log" | xargs grep ERROR
# or
find . -name "*.log" -exec grep ERROR {} +
```

Read more about `grep` and `find`:

```
man grep
man find
```
## Quick-scripting to check state of files in the system

OK, I managed to find hints in the logs as to where were the files, or at least figured out that indeed the script was running - but still the files did not end up where I expected them to be, i.e. not in `/var/app/current/resources`

So then I decided to monitor the contents of this directory, but before that I cleaned it up, in order to get a "clean" deployment

```
rm -f /var/app/current/resources/*
```

And then ran this ad-hoc loop in order to check on the files created on the directory:

```
while true ; do  ls -al /var/app/current/resources ; \
echo "############# `date +%H:%M:%S` ############" ; \
sleep 5  ; done
```

And from the output of the script I learned that my files were indeed placed in the right directory but only for a moment, as right after the "correct" copy, the directory is overwriten by the next ElasticBeanstalk deployment step.

Bottom line - I figured out that the files needed to placed in the temporary source directory prior to the deployment step - and so will end up, eventually in the right place.

Figuring this out was eventually done with the help of the linux commands above, and some reading of documentation.