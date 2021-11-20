# Restore

The first step is to follow the [README](./README.md) to get a K3s installation up and running.

Next, create the `flux-system` namespace for the `sops-gpg` secret, so flux can decrypt and use the secrets in the repo.

```shell
kubectl create namespace flux-system
```

Now import the private key using:

```shell
gpg --import
```

Then, using the `homelab-cluster` gpg key's fingerprint as `$KEY_FP`:

```shell
gpg --export-secret-keys --armor "${KEY_FP}" |
kubectl create secret generic sops-gpg \
--namespace=flux-system \
--from-file=sops.asc=/dev/stdin
```

Now bootstrap flux.

```shell
flux bootstrap github \
  --owner=GiorgioAresu \
  --repository=homelab-gitops \
  --path=cluster/base \
  --personal
```

Tell flux to pull the manifests from Git and upgrade itself with:

```shell
flux reconcile source git flux-system
```

Verify that everything is ok, with:

```shell
flux check
```
