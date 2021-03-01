# Deploy FIO

FIO is a benchmark tool which can used to test storage iops, latency and bandwidth.

From this directory you can build a docker image containing all dependencies needed to run FIO, this can then be used to deploy as a Kubernetes job to test performance of persistent volumes in your cluster.

## Building Docker Image

The Docker image containg all dependencies and configuration for FIO can be found [here](./Dockerfile). There is also an [entrypoint script][./docker-entrypoint.sh] which is called by the docker image. This contains FIO commands to run various tests.

To build the Docker image, run the following:

```bash
docker build -t <docker repo>:<tag> -f Dockerfile .

docker push <docker repo>:<tag>
```

Dont forget to update the [fio-job.yaml](./fio-job.yaml) with your image.


## FIO Tests

FIO has parameters which can be used to simulate different kinds of tests. For full details of FIO, please [read the docs here](https://fio.readthedocs.io/en/latest/fio_doc.html).

The entrypoint has a number of tests it will carry out (you can do a quick test by using the `DBENCH_QUICK` parameter below).

A lot of FIO parameters are baked in to the script as they are set in order to get the desired tests run, such as testing bandwidth (we want larger block sizes), latency (smaller block sizes), read latency (we only want to test read), just as examples. It's recommended to keep these basked in, however some parameters can be passed as environment variables (see below), these can be amount of run time, amount of persistent data to use, mount point for data, whether to just run quick test or all tests.

Please read through the entrypoint script to fully understand how each test is achieved.

## Deploy as a Kubernetes Job

There is an example manifest, to deploy just run:

```bash
# ensure the storageclass "fast-replicated" exists
kubectl create -f fio-job.yaml
```

Once its finished running all tests, check the output logs.

```bash
# get logs output of fio test job and save to log file
kubectl logs fio-test-xxx > results.log
```

Be warned, the default pvc is 50GB and the FIO test uses 10GB, this takes about 4hrs to complete so reduce pvc to around 2GB for quicker tests.

## Parameters

Some of these parameters are baked in to the entrypoint as they should not be changed but ones in upper case are parameters which can be passed via the Job manifest:

| Parameter              | Description                                                                                                 | Default          |
|------------------------|-------------------------------------------------------------------------------------------------------------|------------------|                               
| `FIO_DIRECT`           | If true use non buffered io                                                                                 | `true`           |
| `FIO_SIZE`             | The amount of data used to run the tests                                                                    | `10GB`           |
| `RUN_TIME`             | The amount of time to run the tests for                                                                     | `900s`           |                          
| `FIO_OFFSET_INCREMENT` | Creates an offset amount of data used for sequential read/write testing                                     | `500M`           |
| `gtod_reduce`          | Reduces the get time of day calls                                                                           | `true`           |
| `ramp_time`            | If set, fio will run the specified workload for this amount of time before logging any performance numbers. | `2s`             |
| `ranbd_repeat`         | Seed the random number generator in a predictable way so results are repeatable across runs.                | `false`          |
| `FIO_MOUNTPOINT`    | The Kubernetes mount point to store testing data                                                            | `/data`          |
| `FIO_QUICK`          | Dictates whether this should be a quick test or not, is set to true will only run basic read/write/bw tests
