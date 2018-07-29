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

## Setting up a Kubernetes Service and Deployment

Lets say you want to setup a Web site service running `nginx`

First create a `namespace` so in future you can run multiple environments on the same cluster

Create a file `namespace.yaml`:

```
apiVersion: v1
kind: Namespace
metadata:
  name: mysite-dev
```

Run the following command:

```
kubectl apply -f ./namespace.yaml`
```

Now lets create a Kubernetes Service - i.e. an external Load Balancer in front of the dockers running `nginx` that will be setup momentarily 


Create a file: `service.yaml`

```
apiVersion: v1
kind: Service
metadata:
  name: mywebsite
  namespace: mysite-dev
spec:
  selector:
    app: mywebsite
  ports:
  - protocol: TCP
    port: 80
  type: LoadBalancer
```

Run:

```
kubectl -n mysite-dev apply -f ./service.yaml
```

Check your AWS account for a new Loadbalancer that two instances attached - currently un-healthy

Create a `deployment.yaml` file:

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: mywebsite
  namespace: mysite-dev
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: mywebsite
    spec:
      containers:
        - name: nginx
          image: nginx:1.7.9
        ports:
        - containerPort: 80
```

Run:

```
kubectl -n mysite-dev apply -f deployment.yaml
```

## Verification

1. Check the `nginx` pods have come up by running this command:

```
kubectl -n mysite-dev get pods
```

At first the pods will appear in status `ContainerCreating` and eventually `Running`

E.g. 

```
NAME                       READY     STATUS    RESTARTS   AGE
mywebsite-5fbc6664-b26nf   1/1       Running   0          4m
mywebsite-5fbc6664-n25kl   1/1       Running   0          4m
```

2. Check the AWS Service Load Blanacer has to Instances `InService`

3. Open the External URL of the ELB in port 80 - you should get the default nginx page 