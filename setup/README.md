# k3s cluster

## Node setup (run on each one)

This should probably be done with Ansible, but for now it has to be done manually.

It assumes an Ubuntu Server 20.04 install.


### Hostname

Node names (and hostnames) follow this rule:

kube-*{role}*-*{host}{number}*

where:

- **role**: either master or worker
- **host**: letter identifying the hypervisor or hardware it is running on
- **number**: progressive number for the host

eg. *kube-worker-p2* would be the second worker running on Proxmox

Set the correct hostname if not already set
```shell
export $NODE_NAME=kube-worker-p1
sudo hostnamectl set-hostname $NODE_NAME
sudo vim /etc/hosts # double check it here
```


### Networking

Static IPs or DHCP static mapping, your choice.


### cgroup config

Edit to the grub config in

```shell
sudo vim /etc/default/grub
```

and add 
`cgroup_memory=1 cgroup_enable=memory swapaccount=1` to the `GRUB_CMDLINE_LINUX` line. It will be something like:

```
GRUB_CMDLINE_LINUX="cgroup_memory=1 cgroup_enable=memory swapaccount=1"
```

Then run:

```shell
sudo update-grub
```

### Install requirements

```shell
sudo apt install -y curl nfs-common
```


## Master setup

```bash
curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" INSTALL_K3S_EXEC="--disable traefik --disable servicelb --disable metrics-server" sh -
sudo cat /var/lib/rancher/k3s/server/node-token
```


## Worker setup

Using $K3S_TOKEN from the master

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://kube-master-p1:6443 K3S_TOKEN=${K3S_TOKEN} sh - 
```


## Workstation setup

The following needs to be done on your PC


### Requirements

Install:

- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [flux2](https://fluxcd.io/docs/installation/#install-the-flux-cli)


### Get kube config

```bash
scp kube-master-p1:/etc/rancher/k3s/k3s.yaml ~/.kube/config
```

Open it and replace 127.0.0.1 with the master's ip


## GitHub

Get an Personal Access Token from [GitHub](https://github.com/settings/tokens) and set the envs accordingly:

```
export GITHUB_USER=GiorgioAresu
export GITHUB_TOKEN=new-token
```


## Flux

Now it's the time to bootstrap flux:

```shell
flux bootstrap github \
  --owner=GiorgioAresu \
  --repository=homelab-gitops \
  --path=cluster/base \
  --personal
```

