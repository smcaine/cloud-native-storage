# Kubernetes Cloud Native Storage

Below is a list of Cloud Native storage providers that could be considered as a Kubernetes persistent, fast, local storage solution.


-----

## StorageOS

- **Deployment** 
  - simple to deploy and also has a helm chart. Requires ETCD to be installed as a pre-req, there are different options, recommended is [etcd-cluster-operator](https://github.com/improbable-eng/etcd-cluster-operator)  which also requires cert-manager
  - use crd to create storageos cluster which then creates daomonsets to manage storage on each node
  - file systems on each host need to be in specific format: **/var/lib/storageos/data[0-9]+**
  - no needd to install csi, this is included but you can disable this and install something else

- **Replication**
  - storage can be replicated to other nodes for failover, this is synchronous too so for dr this would be very good.
  - can query resynch of new volumes in logs or using the cli
  - does not support multi cluster replication
  - supports 0 - 5 replicated volumes, however more than 2 replicas is not recommended

- **Topology aware**
  - uses algorithm based on node age, disk space, iops used
  - ensures volumes are replicated on different nodes, can also set up rules using hte cli to create mote advanced placements

- **Data locality**
  - this feature can be enabled to ensure pods are run adjacent to their storage, 2 options: strict/prefered

- **Block**
  - readwriteonce - this wil created block storage on a host which can also be synchronously replicated

- **NFS**
  - can support nfs (although this is not its primary purpose), just create fast storageclass with readwritemany and StoageOS will create this

- **Storage provisioning**
  - 2 options
    - mount new data volume on host which matches the naming format - storageOS will shard available storage
    - expand currently mounted data volumes
  - thin provisioning only
  - uses LVM

- **Replication**
  - auto synchronous replication
  - interesting to note:
     > All replication traffic on the wire is compressed using the lz4 algorithm, then streamed over tcp/ip to target port tcp/5703. If the master volume is lost, a replica is promoted to master (D2 or D3 above) and a new replica is created and synced on an available node (Node 2 or 4). This is transparent to the application and does not cause downtime.

- **DR & backups**
  - requires velero to store off cluster
  - can remove nodes on the fly, seems to be able to handle failover quite well compared to other solutions looking at docs

- **Security?**
  - storage AES encrypted
  - integrates with rbac for multi tenancy access control

- **Requires license**
  - need to consider costs compared to say oss (ceph, linstor, longhorn) or other products (portworx, robin)


-----

## Portworx

- **Deployment**
  - Very simple to install and works similar with an etcd backend and comes shipped with a csi

- **Migrate between clusters**
  - has [Stork](https://docs.portworx.com/portworx-install-with-kubernetes/storage-operations/stork/) (Portworx' scheduler) which can replicate large amounts of data (default max is 100GB but can be configured for more)

- **Asynchronous dr**
  - this again uses stork & px-motion to asynchronoursly replicate data to different k8s cluster

- **Topology aware**
  - can add labels to ensure replica volumes are in separate zone/racks.. this can help speed up maintenance tasks, node replcaements

- **Hyperconvergance**
  - uses stork to score nodes for scheduling pods to same node as volume data

- **Replication**
  - auto synchronous replication of data volumes withing the cluster

- **Shared storage options available**
  - object store
  - nfs

- **Security?**
  - **RBAC**
    - Suports rbac for multi-tenancy access control
    - encryption offerings AES-256

- **Storage provisioing**
  - thin provisioning by defailt but this can be disabled
  - sporeads data across multiple drives

- **Requires license**

- **Performance test**
  - [performance test](https://docs.portworx.com/portworx-install-with-kubernetes/application-install-with-kubernetes/cassandra/measuring-performance/)
    this link shows a real use case of cassandra stress test (instead of using flood.io which is not necessarily the most accurate)


-----

## Rook - Ceph

- **Completely open source**
  - CNCF

- **Deployment**
  - has helm chart as an option to deploy
  - more options to configure compared to previous two, more moving parts
  - have to install csi
  - have to install additional management services (toolbox)
  - configuring block mirroring involves extra steps, you need to enable it when creating the cluster but also configure it on the toolbox service
  - steps are more complicated when compared to other solutions here which just seem to work out the box

- **Storage Clases**
  - object store
  - block storage
  - shared storage
  - need to install csi - either rbd (rwo) or cephfs (rwx)

- **DR & backups**
  - does not have offsite backups, would need to use velero or similar
  - does offer rbd mirroring - asynchronously mirror to another ceph cluster

- **Topology aware**
  - this is limited and we can only specify replicas across hosts or ceph daemons
  - can use erasure coding to fragment data across multiple nodes - performance/latency trade-off

- **Storage provisioing**
  - requires storageclass and pool to be created

- **Security**
  - encryption is available


-----

## Longhorn

- **Completely open source**
  - CNCF
  - **Update** - Rancher now offer support for Longhorn as a standalone product

- **Block storage** - does not have object/nfs storage options

- **Deployment** - 
  - helm/kubectl available
  - requires open-iscsi to be installed on all nodes

- **DR & backups**
  - has auto snapshots and can backup to external storage (s3) but no api written in th ecode yet to authenticate to s3 aws

- **Replication**
  - auto synchronous replication
  - **Required**
  - event logs - Does not have an event log for Longhorn orchestration activities (such as replica rebuild).

- **Security** 
  - **does not have encryption** - this is on the roadmap

- **Scheduling** - this is somewhat basic and just uses anit-affinity to avoid placing replicas on same node, it will in future releases have smarter scheduling algorythm CPU, memory, disk capacity and IOPS 


-----

## Robin

- **Enterprise only**

- **Deployment**
  - looking at the steps, there is an awful lot of configuration needed to get this working on each node, unlike other options where they are deplyed in to kubernetss as operators, this involves a lot more steps and would need to be done at bootstrap maybe via ansible

- **Requested demo but yet to hear back**
  - their site does not give much away, just reading blogs it may be worth considering 


-----

## Linstor

- **Completely open source**
  - CNCF member

- **Deployment**
  - has an operator and also has helm chart (helm3), this is heavily in development and lacking documenation but its a WIP, https://github.com/piraeusdatastore/piraeus-operator

  - can use etcd or postgres/mariadb as its backend kv

- **Topology aware**
  - understands failure domains and these can be customised for on-prem (racks/chasis' etc)

**Replication**
  - synchronous, memory synchronous and asynchronous replication options available
  - uses [DRBD](https://github.com/LINBIT/drbd) to provide RAID1 functionality
  > Synchronization is efficient in the sense that DRBD does not synchronize modified blocks in the order they were originally written, but in linear order, which has the following consequences:
    >Synchronization is fast, since blocks in which several successive write operations occurred are only synchronized once. Synchronization is also associated with few disk seeks, as blocks are synchronized according to the natural on-disk block layout. During synchronization, the data set on the standby node is partly obsolete and partly already updated. This state of data is called inconsistent. The service continues to run uninterrupted on the active node, while background synchronization is in progress.

- **DR & backups**
  - has asynchronous capabilities to mirror to remote cluster

- **Security**
  - encryption is available

- **Hyperconvergance**
  - this can be enabled or disabled to determine if pods should run in same nodes as storage


