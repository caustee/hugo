---
date: "2023-01-18T00:00:00Z"
title: How to restore a Kubernetes cluster after IP change
---

# How to restore a Kubernetes cluster after IP change

Recently I had to fix a kubernetes cluster which had its IP addresses changed. The team that changed the IP address on the machines also replaced the hostname in `/etc/hosts`.
However, this is not enough. In order to restore the cluster, you also have to change the IP address in kubernetes cluster manifest files.

```
[root@k8s-master ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
11.11.112.42 k8s-master
11.11.112.43 worker-k8s2
11.11.112.44 worker-k8s3
11.11.112.42 worker-k8s-control
```

First I confirmed the manifest files include the old IP address of the master node (10.10.138.42):
```
[root@k8s-master ~]# cd /etc/kubernetes/manifests/
[root@k8s-master manifests]# grep "10.10.138" *
etcd.yaml:    kubeadm.kubernetes.io/etcd.advertise-client-urls: https://10.10.138.42:2379
etcd.yaml:    - --advertise-client-urls=https://10.10.138.42:2379
etcd.yaml:    - --initial-advertise-peer-urls=https://10.10.138.42:2380
etcd.yaml:    - --initial-cluster=k8s-master=https://10.10.138.42:2380
etcd.yaml:    - --listen-client-urls=https://127.0.0.1:2379,https://10.10.138.42:2379
etcd.yaml:    - --listen-peer-urls=https://10.10.138.42:2380
kube-apiserver.yaml:    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 10.10.138.42:6443
kube-apiserver.yaml:    - --advertise-address=10.10.138.42
kube-apiserver.yaml:        host: 10.10.138.42
kube-apiserver.yaml:        host: 10.10.138.42
kube-apiserver.yaml:        host: 10.10.138.42
```

Then I replaced it with the new IP address and confirmed the change:
```
[root@k8s-master manifests]# sed -i 's/10.10.138.42/11.11.112.42/g' *
[root@k8s-master manifests]# grep "10.10.138" *
[root@k8s-master manifests]# 
```


Finally, let's check the nodes in the cluster. If you don't wait long enough after the IP change, you will still see kubectl unable to connect to the cluster. But give it a couple of seconds and it will be there:

```
[root@k8s-master manifests]# kubectl get nodes
The connection to the server worker-k8s-control:6443 was refused - did you specify the right host or port?
[root@k8s-master manifests]# kubectl get nodes
NAME           STATUS   ROLES                  AGE    VERSION
k8s-master   Ready    control-plane,master   293d   v1.21.9
worker-k8s2   Ready    <none>                 293d   v1.21.9
worker-k8s3   Ready    <none>                 293d   v1.21.9
[root@k8s-master manifests]# kubectl get ns
```

Simple as that.
