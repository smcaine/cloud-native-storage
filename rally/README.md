Summary
=======
Helm chart to run the official Elasticsearch performance verification tool - rally as a kubernetes job.

Usage
=====

Build docker image:

```bash
docker build -t <docker repo>:<tag> -f Dockerfile .

docker push <docker repo>:<tag>
```
**Important to note**

This docker image uses a config file to be used with esrally: `rally.ini`.

In order to use this at scale, for example some rally tests are over 70GB in size, instead of storing this locally, the image will expect a directory called `esrally-data` to be available. The helm chart will create a pvc and mount it to this location.

To deploy esrally:

- first decide which track you would like to run on your elasticsearch cluster (see [here](https://github.com/elastic/rally-tracks) for details on what tracks are availabel and what they do) it's also possibly to create your own custom tracks too (checkout the [docs](https://esrally.readthedocs.io/en/2.0.0/adding_tracks.html))

- Next, just create a **values.yaml** for that track with the track name as the file prefix. Example:

```yaml
image: <repo>/esrally:1.0.3
track: geopoint
targetHosts: "elasticsearch.elasticsearch:9200"
pipeline: benchmark-only
challenge: append-no-conflicts
dataDir: /esrally-data
volClaimName: geopoint-pvc
reportFile: report-geopoint.csv
storageSize: 20Gi
```

- Then deplpy the helm chart using this saved file:

```
$ helm install geopoint ./chart -f geopoint-values.yaml
```

You can check the progrsss of the job by running:

```bash
kubectl get pods -l app=geopoint-elasticsearch-rally --watch
```

These can take anything from 20 - 90 minutes to complete.

