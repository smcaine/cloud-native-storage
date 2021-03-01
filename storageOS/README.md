# StorageOS

## Overview - Architecture

StorageOS is deployed as a set of pods in to kubernetes. Note the Controlplane and Dataplane are deployed as one daemonset.

- **StorageOS Controlplane**
  Responsible for monitoring and maintaining the state of volumes and nodes in the cluster

- **StorageOS Dataplane**
  Responsible for all I/O path related tasks; reading, writing, compression and caching

- **StorageOS Scheduler**
  Responsible for scheduling applications on the same node as applications volume

- **CSI helper**
  Responsible for registering StorageOS with Kubernetes as a CSI driver

- **StorageOS Operator**
  Responsible for the creation and maintenance of the StorageOS cluster

----------

## Deploying to k8s

### Assumptions

- You hosts that you can deploy kubernetes to and storageos
- Your hosts have file systems mounted with the correct format (see below)
- you will need (n/2)+1 for etcd - there are affinity rules on the etcd pods, so keep workers odd with minimum same as etcd replicas, otherwise this will fail.

### Pre-requisites

In order for this to work, the hosts must already have available file systems with the following name format:

**/var/lib/storageos/data/dev[0-9]+**

If more than one exists, storageOS will create storage pools and shard available volumes using lvm.

**ETCD** - this is required for storageOS to run. This is a separate concern but is being addressed by the vendor.
 I have included etcd operator manifests just to get started, best for quick demos: [here](./etcd), however something else such as [https://github.com/improbable-eng/etcd-cluster-operator](https://github.com/improbable-eng/etcd-cluster-operator) is worth looking at for production. Maybe even Consul?

----------

## Steps to install storageOS in to k8s

**TODO** - Consider using Kusomize or Helm to deploy this

Let's first install etcd-operator:

```bash

kubectl create -f ./etcd/namespace
kubectl create -f ./etcd/operator
kubectl create -f ./etcd/cluster
```

Wait for the etcd pods to start runing:

```bash
kubectl get pods -n etcd --watch
```

- Install storage operator, namespace & crds

```bash

kubectl create -f ./storageos-operator

```

- install stroageOS cluster

```bash
kubectl create -f ./storageos-cluster
```

You should see the storage scheduler, storageos daemonsets and csi helper:

```bash
$ kubectl get pods -n kube-system
kube-system          storageos-csi-helper-5d74dfbb84-qkbss                    2/2     Running
kube-system          storageos-daemonset-cr696                                3/3     Running 
kube-system          storageos-daemonset-nzpjj                                3/3     Running 
kube-system          storageos-daemonset-sj9l4                                3/3     Running 
kube-system          storageos-scheduler-6d4bc74586-wclw9                     1/1     Running 
```

- Now we can install a storageclass

Here's an example:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-replicated
provisioner: csi.storageos.com # Provisioner when using CSI
allowVolumeExpansion: true
parameters:
  fsType: ext4
  pool: default
  storageos.com/replicas: "2" # Enforces 1 replica for the Volume
  # Change the Namespace below if StorageOS doesn't run in kube-system
  csi.storage.k8s.io/node-publish-secret-namespace: kube-system       # Namespace that runs StorageOS Daemonset
  csi.storage.k8s.io/provisioner-secret-namespace: kube-system        # Namespace that runs StorageOS Daemonset
  csi.storage.k8s.io/controller-publish-secret-namespace: kube-system # Namespace that runs StorageOS Daemonset
  csi.storage.k8s.io/node-publish-secret-name: csi-node-publish-secret
  csi.storage.k8s.io/provisioner-secret-name: csi-provisioner-secret
  csi.storage.k8s.io/controller-publish-secret-name: csi-controller-publish-secret
```

You can also set this as the default storageclass:

```bash
kubectl patch storageclass fast-replicated -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

### Register license!!!!

```bash
# If you are just testing, run the following command to access the ui:
kubectl port-forward svc/storageos -n kube-system 5705
```

Log in to the ui (grab the storagos secret) and you will need to register with a StorageOS account. Once this is done, you can use up to 5 nodes and 5TiB of data!

----------

# Summary - what's good, what's not?

**Good**

- Performance - most consistent and produced best results when running FIO and pgbench tests

- Easy to implement (as shown above), just needs filesystems to be mounted on local hosts, then deploy the k8s manifests (operator + cluster crd)

- Excellent support, was by far the best level opf support during POC phase

- Nice UI which shows state of cluster, volumes, nodes, mounts

- Creates a volume pool and shards local storage (uses lvm), makes it easy to expand pvc's on the fly, no need to manually configure this

- Handles encryption

- Handles node failures without any downtime

- No fine tuning needed, works out the box

- Uses compression to help reduce noise

- Smart scheduling, will place new volumes and replicas based on utilizations, available storage

- topology aware, can currently use labels but will soon be able to add affinity rules to ensure volumes/pods are placed across failure domains

**Not so good**

- Have to log in to regsiter the license, (I'm not if maybe they can provide an enterprise image instead)

- affinity rules to be more topology aware, currently only able to use labels as a 'hint' but this is roadmapped to be released in next coming weeks

- etcd is a pain to install and a separate concern.. this is being addressed

- lacks some features like asynchronous replication for dr failover to other clusters

- doesn't have its own backup solution, need velero for this

