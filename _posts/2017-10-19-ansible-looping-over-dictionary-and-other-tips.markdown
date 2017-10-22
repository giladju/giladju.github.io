---
layout: post
title:  "Ansible - Task looping over dictionary tip"
date:   2017-10-19 05:14:49 +0200
categories: devops ansible
---
## Ansible tip - Looping over dictionaries, example with AWS Launch Configurations 

### Prerequisites

The tip in this post assumes you are working with Ansible AWS auto-discovery
See the documentation [here](http://docs.ansible.com/ansible/latest/intro_dynamic_inventory.html)

### Scenario

In this example we have a system that works solely with AWS Auto Scale Groups, in effect the list of Launch Configurations (`lcs`) holds the list of services:

```
lcs:
  - role: frontend
  - role: management
  - role: backend
  - role: algo
  - role: zookeeper
``` 

In this example the `frontend` and the `management` services are customer facing and will run behind an ELB, while the `backend` and `algo` services will run internally and with an Application Load Balancer.
The `zookeeper` service will run internally, but not behind any load balancer

So the detailed `lcs` dictionary looks like this:

```
lcs:
  - role: frontend
    subnet: private
    type: t2.micro
    volumes:
      - device_name: /dev/sda1
        volume_size: 8
    groups: [allow_vpc, allow_ssh]
    elb_location: "public-{{ envname }}"
    elb: True
  - role: management
    subnet: private
    type: t2.micro
    volumes:
      - device_name: /dev/sda1
        volume_size: 8
    groups: [allow_vpc, allow_ssh]
    elb_location: "public-{{ envname }}"
    elb: True
  - role: backend
    priority: "1"
    subnet: private
    type: t2.micro
    volumes:
      - device_name: /dev/sda1
        volume_size: 8
    groups: [allow_vpc, allow_ssh]
    alb_location: "private-{{ envname }}"
    alb: True
  - role: algo
    priority: "1"
    subnet: private
    type: t2.micro
    volumes:
      - device_name: /dev/sda1
        volume_size: 8
    groups: [allow_vpc, allow_ssh]
    alb_location: "private-{{ envname }}"
    alb: True
  - role: zookeeper
    subnet: private
    type: t2.micro
    volumes:
      - device_name: /dev/sda1
        volume_size: 8
    zk: True
```

### Ansible Role Tasks looping through the above dictionary

So in order to create the corresponding Launch Configurations the following three loops need to be created:
Note the filtering of the dictionary using `|` and `selectattr`

```
- name: create customer facing services launchconfigurations
  with_items: "{{ lcs|selectattr('elb', 'defined')|selectattr('elb')|list }}"
  ec2_lc:
    name: "LC-{{ envname }}-{{ item.role }}"
    .
    .
- name: create internal services launchconfigurations
  with_items: "{{ lcs|selectattr('alb', 'defined')|selectattr('alb')|list }}"
  ec2_lc:
    name: "LC-{{ envname }}-{{ item.role }}"
    .
    .
- name: create zookeeper launchconfiguration
  with_items: "{{ lcs|selectattr('zk', 'defined')|selectattr('zk')|list }}"
  ec2_lc:
    name: "LC-{{ envname }}-{{ item.role }}"
    .
    .
```

So assuming our Environment name is `staging` we will get 5 Launch Configurations:

* `LC-staging-frontend`
* `LC-staging-management`
* `LC-staging-backend`
* `LC-staging-algo`
* `LC-staging-zokeeper`