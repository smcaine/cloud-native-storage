# Longhorn

## Overview - Architecture

Longhorn has a distributed architecture, lots of microservices running individually in the cluster:

- Longhorn manager - daemonset responsible for createing and managing volumes

- The longhorn manager creates a controller for each volume and replica created called longhorn engine, docs suggest can suport up to 100,000 of these

- CSI Drivers - for provisioning, expanding, attaching, registering nodes

- Longhorn UI


----------

## Deploying to k8s

### Assumptions

- You have hosts that you can deploy kubernetes to and Longhorn
- Your hosts have file systems mounted to **/var/lib/longhorn**
- to add more volumes, you will need to pre-configure this using lvm on the hosts

### Pre-requisites

- ensure iscsi-initiator-utils is installed on all hosts you want to use storageos
- **/var/lib/longhorn** exists on each host
- mount propogation is enabled
- curl, findmnt, grep, awk, blkid, lsblk must be installed.
- ext4, XFS supported

A basic way to bootstrap each worker node would be a userdata script, to install dependancy and create file systems for example:

```bash
#!/bin/bash
yum install iscsi-initiator-utils -y
mkfs.ext4 /dev/sda -F
mkdir -p /var/lib/longhorn
mount /dev/sda /var/lib/longhorn
FS_UUID=$(lsblk -no UUID /dev/sda)
echo "UUID=$FS_UUID       /var/lib/longhorn    ext4    defaults,nofail        0       2" | sudo tee -a /etc/fstab
```

----------

## Steps to install Longhorn in to k8s

The offical steps can be found [here](https://longhorn.io/docs/1.0.0/deploy/install/install-with-kubectl/) but you can also run the following fro mthis directory:

```bash
kubectl create -f longhorn.yaml
```

Longhorn will auto create storageclass but lets creaet one anyway:

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: fast-replicated
provisioner: driver.longhorn.io
allowVolumeExpansion: true
parameters:
  numberOfReplicas: "3"
  staleReplicaTimeout: "2880" # 48 hours in minutes
```

just run the following:

```bash
kubectl create -f ./storageclass .
```

You can also set this as the default storageclass:

```bash
kubectl patch storageclass fast-replicated -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

## Summary - what's good, what's not?

### Good

- completely opensource, though support would be recomended

- Kafka performance was by far the best

- Postgres performance - was 3rd best but decent overall performance when running pgbench

- Easy to implement (as shown above), just needs filesystems to be mounted on local hosts, then deploy the k8s manifests

- Nice UI which shows state of cluster, volumes, nodes, mounts

- No fine tuning needed, works out the box

- Handles its own backups, no need for velero



### Not so good

- No smart scheduling decisions when creating replicas, just places on different node to primary

- no encryption

- lacks some features like asynchronous replication for dr failover to other clusters

- s3 backup api is not complete, does not have authenticated yet

- least mature

- sceptical about containers required per volume/replica3rd

- FIO performance was very poor compared to others, so performance overall is inconsistent

