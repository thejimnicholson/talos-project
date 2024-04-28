# Setting up a Talos Linux kubernetes bare metal cluster

## Dependencies and prerequisites

You will need the following installed on the system you will be working from. 

- talosctl
- kubectl
- 

## Bootstrapping talos 

TBD. Currently, we manually create nodes in Proxmox and boot them from the Talso ISO, then use the Proxmox VM console to configure IP, gateway, hostnames, etc.

Plan is to implement this using ansible and the Talos nocloud disk image. This creates a chicken-and-egg situation; in order to use cloud-init to bootstrap and configure Talos nodes, we need to have the Talos configuration generated, and specifics for each node configured and loaded into a snippet in Proxmox (so that Proxmox's cloud-init generation logic can apply node-specific configurations.) Currently, very little of this is automated, and there isn't even a web UI into the snippets feature of Proxmox yet, so it's faster to do this manually than to figure out how to automate it.)

## Configuring Talos and kubernetes

```sh
export PRIMARY=10.0.96.41
export CP_NODES=(10.0.96.42 10.0.96.43)
export WK_NODES=(10.0.96.44 10.0.96.45 10.0.96.46)

cd talos

# Create the configuration for control plane and worker nodes
talosctl gen config taloscluster https://$PRIMARY:6443
# Configure the primary node
talosctl apply-config --insecure -n $PRIMARY --file controlplane.yaml

# Bootstrap kubernetes, etc, and etc on the primary node
talosctl bootstrap --nodes $PRIMARY --endpoints $PRIMARY --talosconfig=./talosconfig

# Once kubernetes is up, ask Talos for a copy of the kubeconfig file so we can access it
talosctl kubeconfig ./kubeconfig --nodes $PRIMARY --endpoints $PRIMARY

# Now configure the additional control planes and join them to the cluster. This can take a while
# due to leadership elections for various things
for NODE in "${CP_NODES[@]}"; {
  talosctl apply-config --insecure -n $NODE --file controlplane.yaml
}

# Finally, add some worker nodes
for NODE in "${WK_NODES[@]}"; {
  talosctl apply-config --insecure -n $NODE  --file worker.yaml
}
```

## Bootstrapping flux

First, export your GITHUB_TOKEN. Use a token with commit privileges against your github repository. Or use a different git server, see the flux documentation. 

```sh
flux bootstrap github --owner=thejimnicholson --repository=talos-project  --branch=main --path=apps
```

Once flux is installed, deploying additional services to kubernetes is handled via flux.

This currently includes:

- metallb (an implementation of a L2 load balancer for external access to Kubernetes services)
- nfs-external-subdir-provisioner (Creates a storage class that is backed by an NFS server, and manages space allocations within that NFS mount.)
- nginx ingress controller
- prometheus and grafana
- Mitt Dewald's secret generator (generates secrets with values based on a provided spec, useful for secrets only needed within the cluster.)
- Bitnami's sealed secrets controller (Creates secrets from CRDS created using cluster-specific keys, which are safe to store in source control because the values can only be decrypted if you have the cluster-specific key