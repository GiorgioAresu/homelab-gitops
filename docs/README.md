# k3s cluster

## Node setup

Install Ubuntu Server 20.04 LTS on each node. See [Hostname](#Hostname).
Static IPs or DHCP static mapping, your choice.

Then initialize the cluster with [k3s-ansible](https://github.com/k3s-io/k3s-ansible).

In the inventory group vars, set:

```
extra_server_args: "--disable traefik --disable servicelb --kube-apiserver-arg feature-gates=\"MixedProtocolLBService=true\""
```

We need `MixedProtocolLBService=true` to enable the same LoadBalancer service to use multiple protocols.
[See docs for more info](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/).

### Hostname

Node names (and hostnames) follow this rule:

kube-*{role}*-*{id}*

where:

- **role**: either master or worker
- **id**: identifier for the hardware (or hypervisor) it is running on:
  - **name**: if the name is unique, you can just use that, ie: *nuc*.
  - **name-number**: when there are multiple similar nodes, use a letter (or name) and a progressive number.

eg. The second Raspberry worker node could be called *kube-worker-r2*.

> :warning: If you're changing it manually after install, remember to also check /etc/hosts.


### Additional steps

```shell
ansible -i inventory/my-cluster/hosts.ini k3s_cluster -m apt -a "pkg=curl,nfs-common" --become
```


## Workstation setup

The following needs to be done on your PC


### Requirements

Install:

- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [flux2](https://fluxcd.io/docs/installation/#install-the-flux-cli)


### Get kube config

```bash
scp ubuntu@kube-master-p1:~/.kube/config ~/.kube/config
```


## Flux

Get an Personal Access Token from [GitHub](https://github.com/settings/tokens) and set the envs accordingly, then bootstrap flux:

```
export GITHUB_USER=GiorgioAresu
export GITHUB_TOKEN=new-token
flux bootstrap github \
  --owner=GiorgioAresu \
  --repository=homelab-gitops \
  --path=cluster/base \
  --personal
```

## Volume backups

When creating new volumes, you can add them to a group for automatic backup.
Longhorn is configured to perform automatic snapshots and backups for Volumes labeled with `recurring-job-group.longhorn.io/backup=enabled`.

If you're creating a new Volume, you can label it (**NOT** the PersistentVolume):

```shell
kubectl label volume -n longhorn-system {volume-name} recurring-job.longhorn.io/default- recurring-job.longhorn.io/backup=enabled
```
