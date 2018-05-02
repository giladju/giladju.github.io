---
layout: post
title:  "basic setup of kubeadm on Ubuntu servers"
date:   2018-05-02 05:14:49 +0200
categories: devops kubernetes kubeadm kubectl ubuntu docker
---
# `kubeadm` basic setup

## 'Hardware'

* Master - PC with Ubuntu 18.04 LTS
* Node - VirtualBox with Ubuntu 16.04 LTS

## `kubeadm` installation 

On both Master and Node:


As `root`

```
apt-get update
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
```

```
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
```
    
```
apt-get update
apt-get install -y kubelet kubeadm kubectl
```

```
swapoff -a
```

On the Master only:

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

# `kubeadm` initialization  

## reset 

```
sudo kubeadm reset
```

## setting up kubeadm network with flannel 

```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

Should result in something like

```
Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 10.77.64.37:6443 --token yeg7ui.63ypi932kvkwwp11 --discovery-token-ca-cert-hash sha256:d65f12db10f8dcb4ce8339b1771e409b40e74309eda57eb0bf3a201f19ff5af6
```

Run the commands needed to run the cluster:

```
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Add Node

One the Node server (VirtualBox with Ubuntu 16.04 LTS)

```
kubeadm join 10.77.64.37:6443 --token yeg7ui.63ypi932kvkwwp11 --discovery-token-ca-cert-hash sha256:d65f12db10f8dcb4ce8339b1771e409b40e74309eda57eb0bf3a201f19ff5af6
```


# Network setup - flannel

## Apply network - kube flannel

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml
```

## Checking the cluster is up

```
➜ sudo kubectl get nodes
```

Should result in something like 

```
NAME                  STATUS    ROLES     AGE       VERSION
gilad-thinkpad-t470   Ready     master    15m       v1.10.2
gilad-virtualbox      Ready     <none>    12m       v1.10.2
```

```
➜ sudo kubectl get pods --all-namespaces
```

Should result in something like 

```
NAMESPACE     NAME                                          READY     STATUS    RESTARTS   AGE
kube-system   etcd-gilad-thinkpad-t470                      1/1       Running   0          14m
kube-system   kube-apiserver-gilad-thinkpad-t470            1/1       Running   0          14m
kube-system   kube-controller-manager-gilad-thinkpad-t470   1/1       Running   0          14m
kube-system   kube-dns-86f4d74b45-tncrk                     3/3       Running   0          15m
kube-system   kube-flannel-ds-49w8w                         1/1       Running   0          4m
kube-system   kube-flannel-ds-6trvl                         1/1       Running   0          4m
kube-system   kube-proxy-fl8cv                              1/1       Running   0          12m
kube-system   kube-proxy-n4rsv                              1/1       Running   0          15m
kube-system   kube-scheduler-gilad-thinkpad-t470            1/1       Running   0          14m
```

# Sanity test 

## Create simple pod 

Create `yaml` file - e.g. `just-nginx.yaml`

```   
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx-pod
spec:
  containers:
    - name: nginx-container
      image: nginx
```

Run the container 

```
sudo kubectl create -f just-nginx.yaml
```

The describe it

```
➜ sudo kubectl describe pod my-nginx-pod        
```

You should get something like this:

```
Name:         my-nginx-pod
Namespace:    default
Node:         gilad-virtualbox/10.77.64.158
Start Time:   Wed, 02 May 2018 12:08:13 +0300
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           10.244.1.28
Containers:
  nginx-container:
    Container ID:   docker://a75f1acc8984a32a7c09b4ee8c2b037bece4cc446037c26fba0e38d3219c9d45
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:0fb320e2a1b1620b4905facb3447e3d84ad36da0b2c8aa8fe3a5a81d1187b884
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 02 May 2018 12:08:25 +0300
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-rt7c6 (ro)
Conditions:
  Type           Status
  Initialized    True 
  Ready          True 
  PodScheduled   True 
Volumes:
  default-token-rt7c6:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-rt7c6
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age   From                       Message
  ----    ------                 ----  ----                       -------
  Normal  Scheduled              3m    default-scheduler          Successfully assigned my-nginx-pod to gilad-virtualbox
  Normal  SuccessfulMountVolume  3m    kubelet, gilad-virtualbox  MountVolume.SetUp succeeded for volume "default-token-rt7c6"
  Normal  Pulling                3m    kubelet, gilad-virtualbox  pulling image "nginx"
  Normal  Pulled                 2m    kubelet, gilad-virtualbox  Successfully pulled image "nginx"
  Normal  Created                2m    kubelet, gilad-virtualbox  Created container
  Normal  Started                2m    kubelet, gilad-virtualbox  Started container
```

Note the **IP**

Go to to the webserver `http://<IP>`

E.g. [http://10.244.1.28](http://10.244.1.28)
