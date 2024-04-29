# Deploy secret-generator

```sh
flux create hr secret-generator \
  --release-name secret-generator \
  --source=HelmRepository/mittdewald \
  --chart=kubernetes-secret-generator \
  --target-namespace=secret-generator \
  --create-target-namespace=true \
  --interval=10m \
  --export > helmrelease.yaml
```
