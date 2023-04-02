# k3s cluster

## Node setup

Install Ubuntu Server 22.04 LTS on each node.
Static IPs or DHCP static mapping, your choice.

Then initialize the cluster with [k3s-ansible](https://github.com/k3s-io/k3s-ansible).

In the inventory group vars, set:

```
extra_server_args: "--disable traefik --disable servicelb --disable metrics-server"
```


### Additional steps

```shell
ansible -i inventory/my-cluster/hosts.ini k3s_cluster -m apt -a "pkg=curl,nfs-common,open-iscsi" --become
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
kubectl -n longhorn-system label volume {volume-name} recurring-job-group.longhorn.io/default- recurring-job-group.longhorn.io/backup=enabled
```

You can use something like this to do it in bulk:

```shell
# This will label all Volumes named like plex-config-v1, you may want to double-check not to miss any important one
kubectl -n longhorn-system get volume -o name | cut -d/ -f2 | grep -E '([[:alnum:]]+-)+[[:alpha:]]+-v[[:digit:]]+' | xargs -i kubectl -n longhorn-system label volume {} recurring-job-group.longhorn.io/default- recurring-job-group.longhorn.io/backup=enabled
```
