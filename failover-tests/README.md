# Failover Testing

## Overview

The purpose of these tests is to find out how well each storage solution performs when we need to perform maintenance on a node in the cluster or if a node in the cluster suddnely becomes unhealthy.

### Pre-requisites

- Make sure [elasticsearch](../elasticsearch) is deployed before proceeding with any further steps here.
- Make sure you have installed prometheus/grafana, steps can be found [here](../prometheus)
- Ensure you have added the elasticsearch exporter [dashboard](../elasticsearch/dashboard) in to grafana

## Planned failover test

### Run esrally - nyc_taxis

There is a helm chart to run these jobs [here](../rally). Just run the following in that directory:

```bash
helm install nyc . --values nyc-values.yaml
```
Follow these steps:

- Once this is running, wait 30minutes as it takes a while to get started.
- Drain one of the nodes that is hosting one of the es-data pods, make sure you check what node each es-data-* pod is runing. Try to avoid draining grafana and especially prometheus and of course the nyc_taxis job. If you have to get rid of one, just go with grafana.

You will need to run the following command to evict daemonsets adn persistent data:

```bash
kubectl drain <node> --ignore-daemonsets --force --delete-local-data
```
- Monitor activity of cluster, grab screen shots from grafana.

- Get output of logs from the nyc_taxis job, to see how performance was affected.

```bash
kubectl logs nyc-taxi-xxxx
```

### What just happened

Ran esrally
drained node
Failover is almost seamless, have tried a few attempts at this and one time the init container took approx 5 min to complete but mostly always been a few seconds.
**Important** - wait for esrally to download packages and start running and creating indexes. This puts the storage under more stress and gives more work to do.


### Note

Linstor doesnt re-allocate replicas when a node is drained or fails. You can do this manually but this is complicated asyou have to do per pvc adn there is no friendly GUI:

```bash
# Needs to be run on all pvc's:
 linstor resource-definition auto-place pvc-...
```
This gives it an unfair advantage as storageOS does all this for you without any manual input.


## Node failure test

### Run esrally - nyc_taxis

There is a helm chart to run these jobs [here](../rally). Just run the following in that directory:

```bash
helm install nyc . --values nyc-values.yaml
```
Follow these steps:

- Once this is running, wait 30minutes as it takes a while to get started.
- Next, depending on where you have deployed kubernetes, shutdown one of servers. Make sure as before, you check what node each es-data-* pod is running. Try to avoid draining grafana and especially prometheus and of course the nyc_taxis job. If you have to get rid of one, just go with grafana.

- wait 5 minutes, after this you shoujld see pods status change to `terminating`
- the nodes state should also change to `NotReady`

- **If on bare metal** - If no healthchecks have been configured in the cluster, nothing will get terminated as kubernetes does not know what happened to the node.

**Workaround**

- Delete the failed node:

``bash
kubectl delete <node>
```

There is now a manual step to delete the failed node from the storage cluster follow these steps:

**StorageOS**

Access from the gui, select `nodes` and delete the failed node.

**Linstor**

As there is no GUI, run the following:

```bash
kubectl exec linstor-op-cs-controller-0 -- linstor node lost <node>
```

You should now see the pods initializing and after a few minutes everything should be up and running again.

- Check grafana that the cluster is in a healthy state

- Grab the outputs from nyc_taxis job

