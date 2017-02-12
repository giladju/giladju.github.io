---
layout: post
title:  "Setting up SSH Jump server"
date:   2017-02-11 11:14:49 +0200
categories: devops learn tips ssh
---

## Introduction

Having to work with remote servers frequently you will need to gain access to them quickly and easily. But what to do when security contraints prevent access to the server directly from your workstation?

The solution is a Jump server in your data center / cloud network

![]({{ site.url }}/assets/ssh-jump-host.png)


Host jump
    HostName jump.ocdvlp.com
    User giladjudes
    ForwardAgent yes
ssh-add ~/.ssh/bamboo.pem
ssh-add ~/.ssh/id_rsa
ssh-add ~/.ssh/jumphost-key.pem
ssh-add ~/.ssh/test-us-east-1-key.pem
