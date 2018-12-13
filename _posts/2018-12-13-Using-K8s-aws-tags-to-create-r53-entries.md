---
layout: post
title:  "Dynamic creation of Route53 entries for Kubernetes services"
date:   2018-12-13 05:14:49 +0200
categories: devops kubernetes helm kubectl ansible docker
---
# Creating DNS entries to cryptic Kubernets Service AWS ELBs

## Background

When you create a `service` in Kubernets, e.g. on AWS - you end up with a Load Balancer, thing is the names the AWS ELBs get are not human readable:


![]({{ site.url }}/assets/aws-elb-k8s-names.png)

In come the default AWS Tags created for the ELBs:

```
"tags": {
    "KubernetesCluster": "k8s.stg.company.com",
    "kubernetes.io/cluster/k8s.company.com": "owned",
    "kubernetes.io/service-name": "my-namespace/web-app-backend"
},
```

## Overview

Jinja Filtering through a long, complex dictionary is hard enough, Kubernetes adding dots to Tag names does not help. So we will level the ground a bit by adding custom Tags to the ELBs, simple ones. Allowing Ansible to set a fact of all the running services in the Kubernets cluster that require a DNS entry in Route53.  

## Adding custom tags to Kubernetes Service AWS ELBs

By adding the last line in the following code block, two Tags will be added:

| Key         | Value   |
|---          |---      |
| environment | stg     |
| service     | backend |

```
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.service_name }}
  namespace: company-{{ .Values.env }}-app
  annotations:
      service.beta.kubernetes.io/aws-load-balancer-internal: 0.0.0.0/0
      service.beta.kubernetes.io/aws-load-balancer-additional-resource-tags: 'service={{ .Values.service_name }},environment={{ .Values.env }}'
```

That are defined in the `values.yaml` file in the `helm` directory:

```
env: stg
service_name: backend
```

Running `helm install` or `update` will result in two new tags for the AWS ELB:

```
"tags": {
    "KubernetesCluster": "k8s.stg.company.com",
    "kubernetes.io/cluster/k8s.company.com": "owned",
    "kubernetes.io/service-name": "my-namespace/web-app-backend",
    "environment": "stg",
    "service": "backend"
},
```

## Creating Route53 entries with Ansible, based on our new tags

Following are Ansible tasks that get facts from the ELBs and creates CNAME entries accordingly

```
---
- name: get facts regarding ELBS
  ec2_elb_facts:
    region: "{{ region }}" 
  register: all_elbs
- name: get list of services in cluster 
  set_fact:
    list_of_running_services: "{{ all_elbs.elbs | selectattr('tags.KubernetesCluster', 'defined') | selectattr('tags.KubernetesCluster', 'equalto', cluster_name) | selectattr('tags.service', 'defined')  | map(attribute='tags.service') | list }}"
  vars:
    - cluster_name: "k8s.{{ env }}.company.com"
- name: create DNS entry for k8s services
  route53:
    state: present
    zone: "{{ zone_name }}"
    record: "{{ item }}.{{ zone_name }}"
    type: CNAME
    value: "{{ all_elbs.elbs | selectattr('tags.environment', 'defined') | selectattr('tags.environment', 'equalto', env) | selectattr('tags.service', 'equalto', item) | map(attribute='dns_name') | list }}"
    ttl: 30
  with_items: "{{ list_of_running_services }}"
  ```