This was created as follows:

```sh
flux create hr ingress-nginx \
  --release-name=ingress-nginx \
  --source=HelmRepository/ingress-nginx \
  --chart=ingress-nginx \
  --target-namespace=ingress-nginx \
  --create-target-namespace=true \
  --export > helmrelease.yaml
```


helm show values ingress-nginx --repo https://kubernetes.github.io/ingress-nginx