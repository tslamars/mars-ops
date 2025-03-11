# Working notes

- After task bootstrap:talos, apply the onepassword secret store (this is hidden by .gitignore)
```
kubectl create namespace external-secrets
kubectl apply -f ./kubernetes/apps/external-secrets/onepassword/app/onepassword-secret.yaml
```

## Longhorn specific config for Proxmox
Using my other [talos-k8s-iac](https://github.com/tslamars/talos-k8s-iac) template to provision Talos with a dedicated disk for Longhorn storage, the following needs to be added to the generated machine configs:

Both methods below. Either patch, or apply machineconfig directly.

- Patch volumes
```
talosctl patch mc --nodes 10.10.5.180,10.10.5.181,10.10.5.182 --patch @longhorn-volume.patch.yaml
```

- Patch disks:
```
talosctl patch mc --nodes 10.10.5.180,10.10.5.181,10.10.5.182 --patch @longhorn-disk.patch.yaml
```

- Kubelet section: (make sure indentation is correct, otherwise the disk won't appear in Longhorn)
```
extraMounts:
  - destination: /var/lib/longhorn
    type: bind
    source: /var/lib/longhorn
    options:
      - bind
      - rshared
      - rw
```
- Machine, Disks section:
```
disks:
  - device: /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_drive-scsi1
    partitions:
      - mountpoint: /var/lib/longhorn
```

- Apply the configs to the workers
```
talosctl apply-config -n 10.10.5.180 --file talos/clusterconfig/kubernetes-talos-marsworker-00.yaml
talosctl apply-config -n 10.10.5.181 --file talos/clusterconfig/kubernetes-talos-marsworker-01.yaml
talosctl apply-config -n 10.10.5.182 --file talos/clusterconfig/kubernetes-talos-marsworker-02.yaml
```