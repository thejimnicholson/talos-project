# Monitoring

We are using prometheus and grafana from the community helm chart. It comes with reasonable defaults.

For our installation, we are overriding the charts' storage configuration so that we use persistent storage.

```yaml
alertmanager:
  alertmanagerSpec:
    storage:
      volumeClaimTemplate:
        spec:
          storageClassName: nfs-client
          accessModes: ["ReadWriteOnce"]
          resources:
            requests: 50Gi
grafana:
  persistence:
    enabled: true
    storageClassName: nfs-client
    accessModes: ["ReadWriteOnce"]
    size: 50Gi
    finalizers:
      - kubernetes.io/pvc-protection
prometheus:
  prometheusSpec:
    storageSpec:
      storageClassName: nfs-client
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
thanosRuler:
  thanosRullerSpec:
    storage:
      volumeClaimTemplate:
        spec:
          storageClassName: nfs-client
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 50Gi

```

To see what we can overide, grab the values from the repository:

```sh
 helm show values kube-prometheus-stack --repo https://prometheus-community.github.io/helm-charts > values.yaml
```

You can then either edit values.yaml and use it directly in the `flux create` command, or create a (smaller) values.yaml that just overrides the specifics you want to use. Once you have the configuration you want, you can create the HelmRelease for flux with this command:

```sh
flux create hr kube-prometheus-stack \
  --release-name=kube-prometheus-stack \
  --source=HelmRepository/prometheus-community \
  --chart=kube-prometheus-stack \
  --values=./myvalues.yaml \
  --target-namespace=monitoring \
  --create-target-namespace=true \
  --export > helmrelease.yaml
```
