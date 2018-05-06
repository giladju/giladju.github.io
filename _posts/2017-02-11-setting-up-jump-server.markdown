---
layout: post
title:  "Setting up SSH Jump server"
date:   2017-02-11 11:14:49 +0200
categories: devops learn tips ssh
---

## Introduction

Having to work with remote servers frequently you will need to gain access to them quickly and easily. But what to do when security constraints prevent access to the server directly from your workstation?

In many cases the Security department will not allow you to have private keys on your servers in the cloud, so that the user `gilad` on the jump server should not have any private keys:

```
[gilad@jump ~]$ ls .ssh
authorized_keys  known_hosts
```

The solution is ssh forwarding through the Jump server in your data center / cloud network

![]({{ site.url }}/assets/ssh-jump-host.png)

## Adding keys to the ssh-agent

On your PC, do the following:

In order for the `ssh-agent` to handle the forwarding of your keys you will need to `add` them

Add the following to your `~/.bashrc` (or the corresponding init script of your user)

```
ssh-add ~/.ssh/server.pem
ssh-add ~/.ssh/id_rsa
ssh-add ~/.ssh/jumphost-key.pem
ssh-add ~/.ssh/test-key.pem
```

## Configure your ssh client to Forward keys when accessing the jump host

Still on your PC: 

Add the following to the `~/.ssh/config`

```
Host jump
    HostName jump.domain.com
    User gilad # jump_host_user
    ForwardAgent yes
```

## Make sure your public key is placed correctly on the Jump server

To generate a public key from your private key:

```
ssh-keygen -y -f ~/.ssh/jumphost-key.pem
```

You shoud get a public key:

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCQwxmLonT5JrZUklPCm9N2PycyhJiGioNwfMLgsA2OYqI9ndoMj7eNK4yH3r32M4cBFgG8Y3Nw9hLhAXIA2GfuKSiSfdGepAn6Un/zm1j4LwKZGA/1wdekhIL8pmkNdLZU/N4iAdAvZJ3WPFqaLmFlz7t9AuoPodCF7dPFStBPBcxys17GruxhqnCeoXxjs59P1MsOmucu2dU85yfbKDEinVxuHI5mfH+AEm0zB2GZdBnUUs1gFmm7VT743ELINjVGF36zrtQZUj90ZxirQtfhdJrGjW83hrvlY+6ACuGZcuAGiOm0BhT6LTaUHUU4l0AziWTWgbPzEITQyGQ16hmR
```

On the Jump server: 

Place the public key on the servers in the user's home directory `.ssh` folder in the `authorized_keys` file

Place the public keys on the rest of the servers as defined in the this table:

|user|server|public key of|
|---|---|---|
|`gilad`|jump.domain.com|*jumphost-key*|
|`ubuntu`|192.168.0.144|*test-key*|
|`ubuntu`|172.16.1.123|*server*|

## ssh to the Jump server

`ssh jump`

You should get a prompt on the server:

```
[gilad@jump ~]$
```

## ssh onwards to desired server

E.g.

`ssh ubuntu@192.168.0.144`

Will allow you access based on the `test-key` that does **not** exist on the jump server
