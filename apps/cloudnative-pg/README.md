#  cloudnative postgresql operator

```sh
flux create hr cnpg \
  --release-name=cnpg \
  --source=HelmRepository/cloudnative-pg \
  --chart=cnpg \
  --target-namespace=cnpg-system \
  --create-target-namespace=true \
  --export > helmrelease.yaml
```
