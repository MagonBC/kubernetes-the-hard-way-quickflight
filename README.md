Introduction
============

Welcome to a quick-start to install your k8s cluster faster!

To properly init your first cluster, you need one releavent page which is: the official project documentation [https://kubernetes.io/docs](https://kubernetes.io/docs)

Why is a hard way ?
===================
Because you will setup a cluster from scratch!

Kubernetes is a project offering a lot of tools.

The hard way consist of: putting all pieces together to build your cluster.

Your first try to bootstrap a cluster !
=======================================

The kubeadm command will [boostrap a k8s cluster](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/) for you.

Download it from [this page](https://kubernetes.io/releases/download/)

Then you run the command on your PC/Laptop or any other Linux based system:
```
master:/app/k8s/bin # kubeadm init

I0602 11:00:13.928085    9376 version.go:255] remote version is much newer: v1.30.1; falling back to: stable-1.22

[init] Using Kubernetes version: v1.22.17

[preflight] Running pre-flight checks

[preflight] WARNING: Couldn't create the interface used for talking to the container runtime: docker is required for container runtime: exec: "docker": executable file not found in $PATH

[WARNING Firewalld]: firewalld is active, please ensure ports [6443 10250] are open or your cluster may not function correctly

[WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'

error execution phase preflight: [preflight] Some fatal errors occurred:

[ERROR Swap]: running with swap on is not supported. Please disable swap

[preflight] If you know what you are doing, you can make a check non-fatal with --ignore-preflight-errors=...

To see the stack trace of this error execute with --v=5 or higher
```
Every filled preflight requirement is a step, on the hard way, to get your Kuberntes cluster ready!

The minimal resources
=====================
A minimal lab could be:

![img.png](minimal_cluster.png)

1. The control plane is installed on a 8GB memory/Intel-i5 CPU laptop (Opensuse LEAP 15.5)
2. The worker is installed on a Virtualbox VM (Rocky 9). 

The VM must have two NICs as described. One for local connexion and a NAT to connect to Internet (docker registry ...)

Deploy the lab
==============

```
$ cd ansible
$ make prepare
$ ansible-playbook -i inventories/lab install.yml 
$ kubeadm token create --print-join-command # on master
$ kubeadm join 192.168.56.1:6443 --token y2hoim.yng8ms8ysox4ibln --discovery-token-ca-cert-hash sha256:21b7d76...f0fc # on worker
```

Troubleshooting
===============

If you don't have a lot of resources in your PC like me, non kube-system pods will most probably be hard evicted:
You can work around this by updating hard eviction thresholds in /var/lib/kubelet/config.yaml like this:

```
# default: 15%, 100Mi, 10%, 5%
evictionHard:
  imagefs.available: 5%    
  memory.available: 20Mi
  nodefs.available: 5%
  nodefs.inodesFree: 5%
```

Reference: https://kubernetes.io/docs/concepts/scheduling-eviction/node-pressure-eviction/

Examples
========

Debug DNS Resolution
--------------------

A tutorial on how to create a dnsutils Pod and begin manipulating CoreDNS name resolution: https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/

Install a local Nginx
---------------------

A kubernetes service has by default a cluster IP allowing it to be accessed from all pods inside a cluster.

The manifest [nginx.yaml](https://github.com/MagonBC/kubernetes-the-hard-way-quickflight/blob/main/ansible/roles/kubernetes/files/examples/nginx.yaml) show how to setup an Nginx server and externalise it
using a kube service:

```
$ su - kube
kube@master:~> kubectl apply -f examples/nginx.yaml
kube@master:~> kubectl get services -o wide
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE   SELECTOR
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP    24h   <none>
nginx-svc    ClusterIP   10.110.126.201   <none>        8080/TCP   42m   app=nginx
kube@master:~> nc -vz 10.110.126.201 8080
Connection to 10.110.126.201 8080 port [tcp/http-alt] succeeded!
```

Testing High Availability
=========================

We can extend this lab to test [HA feature](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/). But we need to add some additional node VMs and install a load balancer. We can use the second
[inventory/ha](https://github.com/MagonBC/kubernetes-the-hard-way-quickflight/tree/main/ansible/inventories/ha) and the 
[install_ha](https://github.com/MagonBC/kubernetes-the-hard-way-quickflight/blob/main/ansible/install_ha.yml) script to achieve this goal !

Nice !