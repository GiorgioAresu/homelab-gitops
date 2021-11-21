# Homelab Kubernetes Cluster

GitOps (Flux2) managed Kubernetes (K3s) cluster


## Setup
The cluster is currently running on Ubuntu 20.04 VMs running on Proxmox.

See [setup](setup/README.md) for more info and scripts.


## What is this? How does it work?

This repository contains the entire K8s cluster configuration, via manifests and CRDs that Flux2 uses to syncronize the Kubernetes cluster to the configuration expressed here.

It constantly polls the repo and checks that the cluster state is consistent. If it's not, it does what it needs to make it consistent again.


## Encrypt a secret

Create a secret manifest as you normally would, then encrypt it with:

```shell
sops --encrypt --in-place secret.yaml
```
