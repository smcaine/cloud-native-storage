# Benchmarking PostgreSQL

This example shows how PostgreSQL can be benchmarked on
Kubernetes using PGBench. 

The files in this directory create a PostgreSQL statefulset and a PGBench pod to run the tests.

## Deploy

Run the following in this directory:

```bash
kubectl create -f .
```
Once this is done you can check that a postgres pod is running

```bash
$ kubectl get pods -w -l app=postgres
   NAME           READY    STATUS    RESTARTS    AGE
   postgres-0     1/1      Running    0          1m
```


Exec into the pgbench container and run pgbench, this will run quite a big test:

```bash
$ kubectl exec -it pgbench -- bash -c '/opt/cpm/bin/start.sh'
```

For smaller tests you can just comment out the below in the pgbench pod manifest:

```yaml
      # - name: PGBENCH_CLIENTS
      #   value: "100"
      # - name: PGBENCH_JOBS
      #   value: "4"
      # - name: PGBENCH_TRANSACTIONS
      #   value: "10000"
```

Once you run the tests, you can store result here [pgbench-result](./pgbench-results).