---
layout: post
title:  "Setting up an AWS VPC with Ansible"
date:   2016-08-29 11:14:49 +0200
categories: devops ubuntu git annex
---

## Credits

This post is based on this [page](http://blog.halberom.co.uk/ansible_pub_priv_vpc.html) by [Gerard Lynch](https://www.linkedin.com/in/gerard-lynch-64066261)

## General

In the following `ansible` playbook a VPC is set up with a NAT vm to be used by servers in the Private subnet and a single Web Server vm in the Public subnet

## Prerequisites

**Install Python package `boto`**

```
sudo pip install boto
```

**AWS Key and Secret are added to the Environment Variables**

Add

```
export AWS_ACCESS_KEY_ID=AAAAAAAAAAAAAAAAAAAA
export AWS_SECRET_ACCESS_KEY=aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
```

To your `~/.bashrc`

## Basic `ansible` directory structure

```
  ├── ansible.cfg
  ├── ansible_plugins
  │   └── filter_plugins
  │       └── get_subnets.py
  └── vpc.yml
```

## Ansible Plugin `get_subnets.py`

```
# ansible_plugins/filter_plugins/get_subnets.py
from jinja2.utils import soft_unicode

def get_subnets(value, tag_key, tag_value, return_key='id'):
    # return an attribute for all subnets that match
    subnets = []
    for item in value:
      for key, value in item['resource_tags'].iteritems():
        if key == tag_key and value == tag_value:
          subnets.append(item[return_key])

    return subnets

class FilterModule(object):
    ''' Ansible core jinja2 filters '''

    def filters(self):
        return {
            'get_subnets': get_subnets,
        }
```

**`ansible.cfg`**

```
[defaults]
filter_plugins = ansible_plugins/filter_plugins
```

## The playbook `vpc.yml`

```
{% raw %}
---
# play.yml
- hosts: localhost
  vars:
    vpc_list:
      - region: us-east-1
        state: present
        cidr_block: 10.1.0.0/16
        resource_tags:
          Environment: dev 
        internet_gateway: True
        subnets:
          - cidr: 10.1.0.0/24
            az: us-east-1a
            resource_tags: { "Name": "dev_public", "Environment": "dev", "Tier": "public" }
          - cidr: 10.1.100.0/24
            az: us-east-1a
            resource_tags: { "Name": "dev_private", "Environment": "dev", "Tier": "private" }
    nat_list:
      - region: us-east-1
        keypair: our_ansible_key
        instance_type: "t2.small" 
        image: "ami-b0210ed8" # amzn-ami-vpc-nat-hvm-2015.03.0.x86_64-ebs
        instance_tags: { "Name": "dev_nat", "Environment": "dev" }
        exact_count: 1
        count_tag: { "Name": "dev_nat" }
    webserver_list:
      - region: us-east-1
        keypair: our_ansible_key
        instance_type: "m4.4xlarge" 
        image: "ami-fce3c696" # Ubuntu Server 14.04 LTS (HVM), SSD Volume Type
        instance_tags: { "Name": "Web_Server", "Environment": "dev" }
        exact_count: 1
        count_tag: { "Name": "Web_Server" }

  tasks:

    - name: process vpc
      ec2_vpc:
        region: "{{ item.region }}" 
        state: "{{ item.state }}" 
        cidr_block: "{{ item.cidr_block }}" 
        resource_tags: "{{ item.resource_tags }}" 
        internet_gateway: "{{ item.internet_gateway }}" 
        subnets: "{{ item.subnets }}" 
      with_items: vpc_list
      register: ec2_vpc_out

    - name: ssh-ec2-group
      ec2_group:
        description: HTTP and SSH only from office
        name: open https and ssh
        vpc_id: "{{ ec2_vpc_out.results.0.vpc.id }}"
        region: "{{ ec2_vpc_out.results.0.vpc.region }}"
        rules:
        - proto: tcp
          from_port: 22
          to_port: 22
          cidr_ip: 80.80.80.80/32 # Our office IP
        - proto: tcp
          from_port: 80
          to_port: 80
          cidr_ip: 0.0.0.0/0
        - proto: tcp
          from_port: 443
          to_port: 443
          cidr_ip: 0.0.0.0/0
      register: ec2_sec_group

    - name: nat instance
      ec2:
        region: "{{ item.0.region }}" 
        keypair: "{{ item.0.keypair }}" 
        instance_type: "{{ item.0.instance_type }}" 
        image: "{{ item.0.image }}" 
        instance_tags: "{{ item.0.instance_tags }}" 
        exact_count: "{{ item.0.exact_count }}" 
        count_tag: "{{ item.0.count_tag }}" 
        vpc_subnet_id: "{{ item.1.subnets | get_subnets('Tier', 'public') | first }}"
        assign_public_ip: yes
        wait: yes 
        wait_timeout: 500 
      with_together:
        - "{{ nat_list }}" 
        - "{{ ec2_vpc_out.results }}"
      register: ec2_nat_out

    - name: webserver ec2 instance 
      ec2:
        region: "{{ item.0.region }}" 
        keypair: "{{ item.0.keypair }}" 
        instance_type: "{{ item.0.instance_type }}" 
        image: "{{ item.0.image }}" 
        instance_tags: "{{ item.0.instance_tags }}" 
        exact_count: "{{ item.0.exact_count }}" 
        count_tag: "{{ item.0.count_tag }}" 
        vpc_subnet_id: "{{ item.1.subnets | get_subnets('Tier', 'private') | first }}"
        assign_public_ip: yes
        group: "{{ ec2_sec_group.group_id }}"
        wait: yes 
        wait_timeout: 500 
      with_together:
        - "{{ webserver_list }}" 
        - "{{ ec2_vpc_out.results }}" 
      register: ec2_webserver_out

    - name: update vpc routing tables
      ec2_vpc:
        region: "{{ item.0.region }}"
        state: "{{ item.0.state }}"
        cidr_block: "{{ item.0.cidr_block }}"
        resource_tags: "{{ item.0.resource_tags }}"
        internet_gateway: "{{ item.0.internet_gateway }}"
        subnets: "{{ item.0.subnets }}"
        route_tables:
          - subnets: "{{ item.0.subnets | get_subnets('Tier', 'public', 'cidr') }}"
            routes:
              - dest: 0.0.0.0/0
                gw: igw
# Place holder for setting up the Private network
#          - subnets: "{{ item.0.subnets | get_subnets('Tier', 'private', 'cidr') }}"
#            routes:
#              - dest: 0.0.0.0/0
#                gw: "{{ ec2_nat_out.results.0.instance_ids[0] }}"
      with_together:
        - "{{ vpc_list }}"
        - "{{ ec2_webserver_out.results }}"
        - "{{ ec2_nat_out.results }}"
{% endraw %}
```