
General prep:

 * Adjust the Git repo url in `kubernetes/apps/flux-system/flux-instance/app/helmRelease.yaml`
 * Adjust the domain name in `kubernetes/settings/configMap.yaml`
 * Adjust the base folder of backups in `kubernetes/components/volsync-nfs-backup/secret.yaml`
 * Adjust CiliumLoadBalancerIPPool in `kubernetes/apps/kube-system/cilium/app/networking.yaml`
 * Adjust zfs pool creation in `.justfile`

# Creating the cluster itself

This all needs to be redone

## Bootstrapping

```sh
just go
```
