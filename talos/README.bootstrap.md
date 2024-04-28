# Bootstrapping talos

```sh
export PRIMARY=10.0.96.41
export CP_NODES=(10.0.96.42 10.0.96.43)
export WK_NODES=(10.0.96.44 10.0.96.45 10.0.96.46)

talosctl gen config taloscluster https://$PRIMARY:6443
talosctl apply-config --insecure -n $PRIMARY --file controlplane.yaml
talosctl bootstrap --nodes $PRIMARY --endpoints $PRIMARY \
  --talosconfig=./talosconfig
talosctl kubeconfig ./kubeconfig --nodes $PRIMARY --endpoints $PRIMARY

for NODE in "${CP_NODES[@]}"; {
  talosctl apply-config --insecure -n $NODE --file controlplane.yaml
}


for NODE in "${WK_NODES[@]}"; {
  talosctl apply-config --insecure -n $NODE  --file worker.yaml
}
```
