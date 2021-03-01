# Step by step guides - Cloud Native Storage

Table of Contents
-----------------

-   [Introduction](#introduction)
-   [How to Deploy Cloud Native Storage](#how-to-deploy-cloud-native-storage)
    -  [Pre-requisites](#pre-requisites)
    -  [Let's Get Started](#lets-get-started)
-   [Observability](#observability)
-   [FIO Tests](#fio-tests)
-   [Kafka](#kafka)
-   [Elasticsearch](#elasticsearch)
-   [ESRally](#esrally)
-   [Postgres](#postgres)
-   [Failover Tests](#failover-tests)
-   [Conclusion](#conclusion)

## Introduction

We previously wrote a [whitepaper](../whitepaper) which ran through, in high level, comparisons between some of the leading Cloud native Storage Solutions for Kubernetes.

This page contains a list of step by step guides to get up and running with Cloud Native storage, as well as examples on how to test performance and also scrape and view metrics using tools like Prometheus and Grafana.

## How to Deploy Cloud Native Storage

### Pre-requisites

Firstly some assumptions:

- You are able to deploy a Kubernetes Cluster

- You have master/worker nodes (1 master and 3 - 4 worker nodes would be ideal to carry out all tests)

- We used [wksctl](https://github.com/weaveworks/wksctl) to deploy to bare metal hosts in to Packet, however, you can use any cloud providers, or virtual machines. The worker hosts should have additional storage attached, as its not recommended to use system disks for storage.

- You are familiar with Helm


### Let's get started

Some of the solutions we will use require filesystems to be created and mounted on each worker node in the cluster. It is a good idea to bootstrap this when deploying the worker nodes. In the examples provided, we have added a **Userdata** bash script which will add this on each node.

Click on the following links for step by step guide on how to deploy each storage solution:

- [Deploying StorageOS](../storageOS)

- [Deploying Linstor](../linstor)

- [Deploying Longhorn](../longhorn)

- [Deploying Portworx](../portworx)

----------

## Observability

It's important that we can get a good undertstanding of how the cluster is performing. Adding Prometheus means we can scrape metrics and view them in Grafana.

This can be cluster metrics, Kubernetes based metrics, cpu, memory, network, disk io, we can dig a little deeper by adding application based metrics. This will help us understand why some solutions performed better than others and also check if the cluster is healthy so that we can validate if the tests are fair and accurate.

To deploy Prometheus Operator, follow this guide: [Deploy Prometheus Operator](../prometheus).

----------

## FIO Tests

FIO is a benchmarking tool which is used to test iops, latency and bandwidth. Full details can be found here [fio: read the docs](https://fio.readthedocs.io/en/latest/fio_doc.html)

You can find steps to build a docker image which has fio installed, as well as an entrypoint script which is used to run various benchmark tests, along with step by step guide on how to view log outputs from fio [here](../fio-test).

----------

## Kafka

Kafka is a distributed streaming/mesaging platform which requires high throughput and low latency to handle large amounts of data feeds. This would be an ideal candidate for Cloud Native storage.

The following guide will run through how to deploy Kafka in to Kubernetes, as well as providing a way to scrape all metrics and view them in Grafana and of course some examples of how to run performance tests and view the results:

- [Getting started with Kafka](../kafka)

----------

## Elasticsearch

Elasticsearch is a distributed analytics/search engine, it requires low latency, fast performance, scalable and highly resilent. This also makes it the perfect candidate for our local storage use case.

Follow this guide to [get started with Elasticsearch](../elasticsearch), it will run through deployment of Elasticsearch in to Kubernetes, how to scrape metrics and view them as dashboards in Grafana.

### ESRally

For performance testing, esRally is a great tool as it uses real world data which is then created in the cluster and then runs various benchmark tests.

Once you have Elasticsearch deployed and you can view dashboards in Grafana, follow [this guide](../rally) on how to use esRally in yoru cluster. There is a guide on how to buld a docker image and a example Helm chart which can be used to deploy a job that runs all the tests.


----------

## Postgres

We deployed Postgres as a database backend and tested its performance using PGBench.

This [guide](../postgres) will run through how to deploy Postgres as a statefulset in to a Kubernetes cluster, as well as deploying PGBench as a Kubernetes job to run through reading/writing data and viewing the results.

----------

## Failover Tests

It made most sense to run failover tests whilst running [Elasticsearch](../elasticsearch) and [ESRally](../rally), although [Kafka](../kafka) would also work well.

The idea was to have tests that took a decent amount of time to perform and during the tests, we could perform worker node failovers. This was split into 2 scenarios:

- **Planned Failover** - Elasticsearch was deployed, esRally tests were run and then we drained a worker node and monitored performance and compared results.

- **Unplanned Failover** - as above but this time a worker node was ungracefully powered off, we then monitored performance and compared results

For results of planned and unplanned, please visit this page [failover-tests](../failover-tests)

----------

## Conclusion

Hopefully this article has been useful and will help you more in choosing the right storage solution for your Kubernetes platform.

In these examples, there were 2 that showed more consistency across all tests and just a bit more mature in their current state. These were [Linstor](../linstor) and [storageOS](../storageOS), however when it came to kafka, [Longhorn](../longhorn) was definitely a different league as far as performance.

So it really depends on your use cases which one to go for but storageOS seemed to offer the all round package as far as maturity, ease of use, support and performance, it was so consistent.

I would recommend always testing these in your environment to see how the results fair and definitely reach out to the vendors who will be able to offer further support and help fine tune everything for you.