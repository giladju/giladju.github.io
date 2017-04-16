---
layout: post
title:  "Installing git annex on Ubuntu 14.04"
date:   2016-09-20 11:14:49 +0200
categories: devops ubuntu git annex
---

# The problem

`git annex` is a bit tricky, and I found that things have improved in later versions of the package, but the latest version available by default for Ubuntu 14.04 is from 2014 when in fact the fixes I needed were in the version from 2015
So, following are instructions on how to install the latest version, enjoy 

# Prepare for installing

    echo "deb http://mirror.aarnet.edu.au/pub/neurodebian data main contrib non-free" | sudo tee /etc/apt/sources.list.d/neurodebian.list
    echo "deb http://mirror.aarnet.edu.au/pub/neurodebian trusty main contrib non-free" | sudo tee -a /etc/apt/sources.list.d/neurodebian.list 
    sudo apt-key adv --keyserver hkp://pgp.mit.edu:80 --recv-keys 0xA5D32F012649A5A9
    sudo apt update

# Verify you are about to install the latest version

    apt-cache show git-annex-standalone

You should get:

    Package: git-annex-standalone
    Source: git-annex
    Version: 6.20160307+gitgb095561-1~ndall+1

**Note:** the version is 6.**2016**

# Install 

    sudo apt install -y git-annex-standalone