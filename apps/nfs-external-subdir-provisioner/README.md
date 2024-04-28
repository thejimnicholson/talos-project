
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
