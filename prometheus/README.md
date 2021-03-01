# Prometheus

Prometheus is a great tool to use for monitoring/observability in your Kubernetes cluster. It is open source and has an operator which comes pre-configured with lots of dashboards and exporters to help get you started.

## Steps to install

### Pre-requisites

We will use the offical [Helm chart](https://github.com/helm/charts/tree/master/stable/prometheus-operator) to install Prometheus.

### Steps

It is important to install the crds for the following:

- **Alert Manager**
- **Pod Monitors**
- **Prometheus Alert Rules**
- **Service Monitors**
- **Thanos Ruler (not really needed for testing)**

- Install the crds in this directory first!

```bash
kubectl create -f .
```

Now you are ready to install the helm chart:

```bash

helm repo add incubator http://storage.googleapis.com/kubernetes-charts-incubator

helm repo update
# This option stops prometheus operator from trying to deploy crd's, it's a workaround and may well be fixded when you try to run this:
helm install prometheus stable/prometheus-operator --set prometheusOperator.createCustomResource=false

```

