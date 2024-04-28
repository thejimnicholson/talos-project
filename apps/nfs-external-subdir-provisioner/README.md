# Deploy nfs-subdir-external-provisoner via flux

This was created as follows:

```sh
flux create hr nfs-external-subdir-provisioner \
  --release-name=nfs-subdir-external-provisioner \
  --source=HelmRepository/nfs-subdir-external-provisioner \
  --chart=nfs-subdir-external-provisioner \
  --values=./values.yaml \
  --target-namespace=nfs-provisioner \
  --create-target-namespace=true \
  --export > helmrelease.yaml
```

values.yaml contains:

```yaml
nfs:
  server: "10.0.96.14"
  path: "/mnt/data/k8s/talos"
```
