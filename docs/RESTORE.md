# Restore

The first step is to follow the [README](./README.md) to get a K3s installation up and running.

Next, create the `flux-system` namespace for the `sops-age` secret, so flux can decrypt and use the secrets in the repo.

```shell
kubectl create namespace flux-system
```

Restore the age key file to `~/.config/sops/age/keys.txt` and import it with:

```shell
cat ~/.config/sops/age/keys.txt |
kubectl create secret generic sops-age \
  --namespace=flux-system \
  --from-file=age.agekey=/dev/stdin
```

Now bootstrap flux.

```shell
flux bootstrap github \
  --owner=GiorgioAresu \
  --repository=homelab-gitops \
  --path=cluster/base \
  --personal
```

Verify that everything is ok, with:

```shell
flux check
```

## Volume backups

Setup longhorn with a recurring backup (optionally, snapshot too) job for group `backup`, so that important data gets backed up automatically.
