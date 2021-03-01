# Portworx

## Overview - architecture

Portworx by design is distributed in to many separate services, (similar to Longhorn and Linstor).

When deplopyed you will see daemonsets, csi's, api controllers, lighthouse (gui), stork and etcd (it's posible to use built in or byoetcd)

## Steps to install portworx

- Deploy prometheus crd's first. The manifests for portworx contain a servicemonitor crd and wil fail to deploy unless these exist.

- Create an account on Portworx [site](https://central.portworx.com/specGen/wizard) and run through the online wizard to download the yaml file to deploy.

- Deploy Portworx manifest (make sure you unlink license if reinstalling!!!)

- Create storageclass and patch to make further deployments easier:

```bash
kubectl patch storageclass fast-replicated -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

----------

## Summary - what's good, what's not?

### Good

Easy to implement due to excellent documentation and I nice interactive wizard which is used to confugure your environment.

They offer a feee version up to 5TiB and 5 nodes, which is great for smaller environments to use.

Great community support via slack.

Has all features such as dr, backup, encryption, ui, smart scheduling

You can use affinity rules to dictate where volumes and replicas are placed

Very easy to implement, no pre-req's on host.

The most mature product on offer, nothing is "in the works" or "roadmapped", everything you need and want is ready to go.

Can auto expand volumes.


### Not so good

Is the most expensive by far, although it is worth checking at the time to see what prices you are quoted.

The performance does not appear to justify the price.

