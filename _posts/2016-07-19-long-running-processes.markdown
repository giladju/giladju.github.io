---
layout: post
title:  "Executing a few long running processes"
date:   2016-07-19 11:14:49 +0200
categories: devops ubuntu processes
---


Working with Big Data applications one has to crunch a lot data which makes the prospect of long running processes hanging-up daunting.
Below is a script that uses both `nohup` & `pidof`
`nohup` in order to run commands immune to hangups, and `pidof` follow up on PIDs of the processes later

---

```
#! /bin/bash
if [[ $# -eq 0 ]] ; then
    echo 'Usage: runsome.sh "command string" <#> /log/dir'
    echo 'Where <#> is number of processes you want to run'
    echo 'Optional: 3rd argument is directory where logs will be writen - default: /tmp'
    echo 'Optional: 4th argument is Log prefix in case of multiple runs'
    exit 0
fi

thecmd=$1
procnum=$2
logdir=$3
prefix=$4

mkdir -p $3/tmp

for i in $(seq 1 $procnum)
    do
	    nohup $thecmd 2>&1 > $3/tmp/${prefix}log$i.log &
done

cmdstr=`echo $thecmd | awk -F" " '{ print $1 }'`

echo "PIDs"
pidof $cmdstr
```