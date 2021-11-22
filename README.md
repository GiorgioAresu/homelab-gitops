# Homelab Kubernetes Cluster

GitOps (Flux2) managed Kubernetes (K3s) cluster


## What is this? How does it work?

This repository contains the entire K8s cluster configuration, via manifests and CRDs that Flux2 uses to syncronize the Kubernetes cluster to the configuration expressed here.

It constantly polls the repo and checks that the cluster state is consistent. If it's not, it does what it needs to make it consistent again.


## :sparkles: Features

Some of the interesting additions running on the cluster are:


### [MetalLB](https://metallb.org/)

Allow LoadBalancer services to get a local IP address, using either DHCP or BGP.


### [NFS Subdir External Provisioner](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner)

Automatic provisioner that use your existing and already configured NFS server to support dynamic provisioning of Kubernetes Persistent Volumes via Persistent Volume Claims.


### [Node Feature Discovery](https://kubernetes-sigs.github.io/node-feature-discovery)

NFD detects hardware features available on each node in a Kubernetes cluster, and advertises those features using node labels.
These labels can then be used to help with scheduling workloads on the most appropriate node.

## :construction_worker: Setup
The cluster is currently running on Ubuntu 20.04 VMs running on Proxmox.

See [setup](setup/README.md) for more info and scripts.


## Encrypt a secret

Create a secret manifest as you normally would, then encrypt it with:

```shell
sops --encrypt --in-place secret.yaml
```
