# Homelab Kubernetes Cluster

GitOps (Flux2) managed Kubernetes (K3s) cluster


## What is this? How does it work?

This repository contains the entire K8s cluster configuration, via manifests and CRDs that Flux2 uses to syncronize the Kubernetes cluster to the configuration expressed here.

It constantly polls the repo and checks that the cluster state is consistent. If it's not, it does what it needs to make it consistent again.


## :sparkles: Features

Some of the interesting additions running on the cluster are:


### [SMB CSI Driver for Kubernetes](https://github.com/kubernetes-csi/csi-driver-smb)

Access SMB shares, also supports dynamic provisioning.


### [Descheduler](https://github.com/kubernetes-sigs/descheduler)

Evicts pods based on different configurable policies.


### [Longhorn](https://longhorn.io)

Cloud native distributed block storage for Kubernetes.


### [Intel GPU Plugin](https://artifacthub.io/packages/helm/k8s-at-home/intel-gpu-plugin)

Allow offloading to Intel GPU for transcoding and such.


### [Kured](https://github.com/kubereboot/kured)

Safely reboot nodes when required by system updates.


### [MetalLB](https://metallb.org/)

Allow LoadBalancer services to get a local IP address, using either DHCP or BGP.


### [NFS Subdir External Provisioner](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner)

Automatic provisioner that use your existing and already configured NFS server to support dynamic provisioning of Kubernetes Persistent Volumes via Persistent Volume Claims.


### [Node Feature Discovery](https://kubernetes-sigs.github.io/node-feature-discovery)

NFD detects hardware features available on each node in a Kubernetes cluster, and advertises those features using node labels.
These labels can then be used to help with scheduling workloads on the most appropriate node.


### [Renovate](https://docs.renovatebot.com/)

Open PR when new charts versions are available.


### [Secret Reflector](https://github.com/emberstack/kubernetes-reflector)

Mantain Secrets and ConfigMaps synced among namespaces.


### [Stakater Reloader](https://github.com/stakater/Reloader)

Watch changes in ConfigMaps and Secrets and do rolling upgrades on Pods with their associated DeploymentConfigs, Deployments, Daemonsets Statefulsets and Rollouts.


### [System Upgrade Controller](https://github.com/rancher/system-upgrade-controller)

General-purpose, Kubernetes-native upgrade controller (for nodes). It introduces a new CRD, the Plan, for defining any and all of your upgrade policies/requirements. A Plan is an outstanding intent to mutate nodes in your cluster.


## :construction_worker: Hardware

The main idea was to have a cheap yet reliable cluster to:
- Self-host various applications.
- Learn gitops.
- Practice after CKAD outside of work (free to explore new interesting stuff and do things by the book).

The desiderata was for it to be as self-contained as possible to allow the main storage server to be shut down if needed (to save power and keep noise down). A small number of applications will still rely on it, be it due to the larger available size or for data accessibility and reliability reasons.

The end result is currently an *arm64* and *amd64* hybrid cluster, with every node running Ubuntu Server 20.04 LTS (or similar distro).
The initial plan was a pure *arm64* cluster, but the unused NUC proved useful for some *amd64*-only images and for specific workloads, such as hardware-accelerated video transcoding.

|       Device       |   Disk Size   | Ram |           Purpose          |
|:------------------:|:-------------:|:---:|:--------------------------:|
| Raspberry PI 4     | 128GB MicroSD | 8GB | k3s Master (embedded etcd) |
| Raspberry PI 4     | 128GB MicroSD | 8GB | k3s Worker                 |
| Intel NUC DC3217BY | 240GB SSD     | 8GB | k3s Worker                 |

See [setup](docs/README.md) for more info and scripts.


## Encrypt a secret

Create a secret manifest as you normally would, then encrypt it with:

```shell
sops --age=*{pubkey}* --encrypt --encrypted-regex '^(data|stringData)$' --in-place secret.yaml
```
