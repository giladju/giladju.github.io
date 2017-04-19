---
layout: post
title:  "Continuous Integration with AWS tools only"
date:   2017-04-10 05:14:49 +0200
categories: devops "Continuous Integration" "AWS Codepipeline"
---

## The Goal

Working in a professional service organization, one might be called to the helm to help out with a task for the common good, in this case setting a build process for a straight forward `node.js` project. 

Being familiar with setting up a build and deploy process with Jenkins, this would be the quickest way to go. But the goal is to learn and bring in to the company some knowhow. So together with the goal of saving a few $ we opted to go with AWS tools.

Presumably it would be straight forward to setup a build and deploy process from git to running server(s) with CodePipeline.

So here are my experiences so far

## Assumptions

As the  saying goes, to *assume* is making an *ass* of *u* and *me* ... 

So I assumed that the process of getting code from [github](github.com) to AWS is pretty similar to the way Jenkins accesses the code, i.e. `ssh`

I also assumed, correctly it seems, that there is a build process and a deployment process. But I'm jumping ahead of myself

## Initial setup - AWS and Github

It was great to see that AWS can pull code from Github.

![]({{ site.url }}/assets/aws-github.png) 

But when I pulled down the list of git repos I can access, I only saw my private repos, not the company's. Not until I was made Admin of the repos `orginization\repo` was I able to see it on the list. 

This will do for now, but truthfully is a show-stopper. Needing a private github account in order to access a git repo is bad enough, but requiring the user be an `admin` is opening a room for error, that probably is a deterrent for a lot of Build Managers, certainly their leads. 

## Moving on - Building

Considering there is not much of a build in this case, i.e. the only build step we need is:

```
tar cvfz build-`date +%Y%m%d-%H%M`.tgz
```

And the resulting artifact would be: `build-*.tgz` 

I thought this step would go smoothly. 

It didn't

It seems we need and *IAM* role, specifically a *Service Role*. My AWS user does not have permissions to create (or list) IAM roles, so I need an Admin to create a role for the build process and then grant my Listing permission on IAM roles. 

There is a lot of to be said regarding limiting access in AWS, but it would be nice to have pre-set IAM Role/Policy that allows a non-admin user to actully complete a CodeBuild - CodeDeploy/Elastic-Beanstalk build/deploy process 

So this is where I am stuck for now, waiting for my friendly Admin to wake up ...

## Building - after getting permissions

Well, only after getting AWS Admin permissions could I continue.

So now the `AWS CodePipeline` has the first two of three Continuous Deployment steps in place:

* Source 
* Build

Took a bit of documentation reading, but in the end, a basic "build" step was put in place that does the basics of packing all the `node.js` code in a zip file

## Deployment 

The Deployment step requires you setup an `AWS ElasticBeanstalk` Application and Environment. Pretty trivial for someone familiar with AWS EB, but might require a bit of tinkering for someone new to the module.

## Conclusion

![]({{ site.url }}/assets/aws-pipeline.png) 

First of all, one has to admit, it works - we now have a Continuous Integration Pipeline up and running, without redundant servers (i.e. Jenkins). 

I.e. completely based on AWS modules, and minimum cost, presumably only paying extra for the Build time (above the cost of the environment servers)

## Open issues

1. The integration with GitHub required my user be an Admin on the specific git repo. Considering this is my private github account that is part of a company Continuous Integration process - this is a big no no
2. The build process is pretty basic and has no cleanup process - i.e. old artifacts will remain in S3 until someone goes in and cleans them up 
3. The "deployment" process in Elastic Beanstalk has no mechanism of "pausing" the environment so as not to waste money in EC2 uptime 




