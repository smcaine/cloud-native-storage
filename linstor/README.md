# Linstor

## Overview - Architecture

Linstor uses DRBD (can be used without using zfs or lvm) to create distributed replicated storage pools. The commercial offering provides a helm chart which can be used to deploy everything needed to get LInstor running in your cluster.

There is a community operator but unfortunately support for this is not provided by Linstor and the docs are not easy to follow, I would strongly recommend sticking with commercial offering.

Once the operator is deployed you can then deploy nodesets and controllers to manage storage and create pvc's. The following services will be depopyed in the cluster:

- linstor-controller - managed the state of the cluster

- linstor-satellite - node agent which runs the commands received fro the controller

- nodeset - daemonset will create stroage pools on each node

- csi controller + csi daemonset

- stork - can use this for advanced scheduling, migration, backups etc


---------

### Steps to Deploy

Official docs can be found [here](https://www.linbit.com/drbd-user-guide/linstor-guide-1_0-en/#ch-kubernetes)

There are no pre-requisites required for the worker nodes, the operator installs everything needed for this to work.

```bash
# label each worker node that will be used for persistent storage:
kubectl label node <node name> linstor.linbit.com/linstor-node=true

# Recommended to register an account with linstor and obtain license to use their official helm chart and images: 
kubectl create secret docker-registry drbdiocred --docker-server=drbd.io --docker-username=<linstor username> --docker-email=<email address registrered with linstor> --docker-password=<linstor password>
```

optional for testing:

`etcd.persistence.enabled=false`

if you want to add etcd hostpath:
```bash
helm repo add linstor https://charts.linstor.io
helm upgrade --install linstor-etcd linstor/pv-hostpath --values pv-values.yaml

#for centos7:

helm repo add linstor https://charts.linstor.io

helm install linstor-op linstor/linstor --set etcd.persistence.enabled=false --set operator.nodeSet.automaticStorageType=LVMTHIN

# or with etcd

helm install linstor-op linstor/linstor --values values.yaml
```

check everything:

https://www.linbit.com/drbd-user-guide/linstor-guide-1_0-en/#s-kubernetes-linstor-interacting

```bash
kubectl exec linstor-op-cs-controller-0 -- linstor storage-pool list
```

Next, create storageclass

```bash

kubectl create -f ./storageclass/replicated.yaml

#set this as default
kubectl patch storageclass fast-replicated -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

----------

## Summary - what's good, what's not?

**Good**

- Performance postgres - the best performance when running pgbench

- Performance - consistent performance on FIO and kafka 

- Completely opensouce, support recommended in order to get the official install packages

- Everything can be deployed using the kubernetes operator adn crd's (commercial only)

- Synchronous replication

- Can be fined tuned to suit the environment

- Extensive documetation

- RDMA compatible -  can run over InfiniBand networks

- Supports LVM, ZFS, can think or thick provision

- can manage backups and dr asynchronous replication

- handles encryption


**Not so good**

- Steeper learning curve compared to others

- Support were slow to reply, especially on slack

- Community offerings not very easy to implement

- Documentation is confusing, the link above was sent by an engineer but i couldnt navigate to it from the site

- No UI
