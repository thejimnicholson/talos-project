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

How to get values to override:

```sh
helm show values ingress-nginx --repo https://kubernetes.github.io/ingress-nginx
```

How to prove ingress is working without changing DNS:

```sh
# Deploy httpd 
kubectl create deployment demo --image=httpd --port=80

# Expose httpd as a service in the cluster
kubectl expose deployment demo

# Create an ingress for the service
kubectl create ingress demo --class=nginx \
  --rule www.demo.io/=demo:80

# Find the external IP (provided by metallb) for the ingress controller
EXT_IP=$(kubectl get service ingress-nginx-controller -n ingress-nginx -o yaml | yq '.status.loadBalancer.ingress[0].ip')

# Ping the ingress controller via curl to see if it works. 
curl --resolve www.demo.io:80:$EXT_IP http://www.demo.io
```

You should see:

```html
<html><body><h1>It works!</h1></body></html>
```
