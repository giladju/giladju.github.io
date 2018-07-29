---
layout: post
title:  "basic setup of kubernetes with kops"
date:   2018-07-29 05:14:49 +0200
categories: devops kubernetes kops kubectl terraform docker
---
# Setting up a Kubernetes cluster with kops

## Preparation

In order to start you will need the following:

- AWS Accout with IAM user that has Admin privleges 
- Route53 Zone  (e.g. dev.mysite.com)
- S3 Bucket (e.g. s3://my-terraform-state)
- SSH Keypair  (e.g. mykey.pub)
- Avialable Subnet CIDR (e.g. 10.60.0.0/16)
- Linux client with the following installed on it:
  - `awscli`
  - `terraform`
  - `kops`
  - `kubectl`

## Setting up the Cluster with `kops`

Run `kops` to genrate the `terraform` code needed to setup the cluster

```
kops create cluster --name=k8s.dev.mysite.com --state=s3://my-terraform-state --dns-zone=dev.mysite.com --out=. --target=terraform --zones="us-east-1a,us-east-1b,us-east-1c" --ssh-public-key="../mykey.pub" --topology private --networking calico
```

Expected output:

```
kops has set your kubectl context to k8s.dev.mysite.com

Terraform output has been placed into .
Run these commands to apply the configuration:
   cd .
   terraform plan
   terraform apply
```

Run the following command:

```
terraform apply
```

To add a `bastion` servers do:

```
kops create instancegroup bastions --role Bastion --subnet utility-us-east-1c --name k8s.dev.mysite.com --state=s3://my-terraform-state
kops update cluster k8s.dev.mysite.com --state=s3://my-terraform-state --yes
```

To figure out the name of the AWS ELB in front on the bustion docker:

```
aws elb --region=us-east-2 --output=table describe-load-balancers|grep DNSName.\*bastion|awk '{print $4}'
```

To access the bastion server:

```
ssh -A admin@`aws elb --region=us-east-2 --output=table describe-load-balancers|grep DNSName.\*bastion|awk '{print $4}'`
```
