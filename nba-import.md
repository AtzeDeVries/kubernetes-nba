# NBA Import

## Overview

There are two ways to put data in a elasticsearch cluster. Restore using
a snapshot or run a import. First we will discuss the snapshot method, then
the import method. The current import requires about 500GB of storage.

## Snapshot

Requirements:

* A snapshot (S3 preferred) repository with a snapshot
* Access to Kibana or HTTP access of Elasticsearch

### Config Repository

The manual is based on kibana. Modifying to curl is easy. First check if
a repository already exists:

```http
GET _snapshot/_all
```

If that's not the case, add a repository:

```http
PUT _snapshot/nba-import-snapshots
{
  "type": "s3",
  "settings": {
    "bucket" : "nba-import-snapshots",
    "region" : "us-east-1",
    "endpoint" : "http://10.233.30.173:9000",
    "access_key" : "changeme",
    "secret_key" : "reallyreally",
    "protocol" : "http",
    "max_retry": "25",
    "max_snapshot_bytes_per_sec": "150mb"
  }
}
```

Check the available snapshots:

```http
GET _snapshot/nba-import-snapshots/_all
```

### Run restore

You should either close the indices you want to restore or delete them. For NBA
deleting them is the easy choice since the snapshots are not incremental. Delete
the indices by:

```http
DELETE /specimen,taxon,multimedia,geoares
```

Then run a restore:

```http
POST /_snapshot/nba-import-snapshots/<snapshot name>/_restore
{
  "indices" : "specimen,taxon,multimedia,geoares"
}
```

Check status of indices with:

```http
GET _cat/indices
```

After restore is done, the number of replicas van be changed to the desired state

```http
PUT /specimen,multimedia,taxon/_settings
{
  "number_of_replicas": 1
}
```

## Import

To do an import go to the folder `import/parajobs`. The import has four phases.
First check if there are any jobs this in k8s. If there are already jobs running
you can delete all runs from the current folder:
`kubectl delete -R -f ./`

Before a import, check the followig things:

* Import config `0-prepare/etl-configmap-nba.propierties`
* ETL versions (docker tag) in the folders 0,1,2
* Source data versions (git tags) in folder 1

### Prepare indices

`kubectl create -f 0-prepare/`
Wait until the job is done. You can check the job status via
`kubectl get jobs`. Or check pod status via `kubectl get pods --watch`

### Import sources

`kubectl create -f 1-source-import/`
This process can take between 2 and 5 hours depending on your import cluster.

### Enrich data

Wait until the import sources is done and start the enrichment of the data with:
`kubectl create -f 2-enrich/`
This will add extra data to all documents. This step takes about 3 hours.

### Postjobs

Wait until the enrichtments are done
`kubectl create -f 3-postjobs`. This job will cleanup all deleted documents
in the indices. This saves space and enchanses query performance.
In kibana you can check the status of the cleanup with `GET _cat/thread_pool` and
check if there are any `force_merge` processes running. You can also check
`GET _cat/indices?v` and check for the NBA indices if the number of deleted
documents is 0.

### Post kibana jobs

Check in kibana if the snapshot repository is available.
Then check in minio if there is enough space available for a snapshot,
otherwise delete an older snapshot.
You can check the current available snaphosts with:

```http
GET _snapshot/nba-import-snapshots/_all
```

And delete one with:

```http
DELETE _snapshot/nba-import-snapshots/<snapshot name>
```

Then you can take a snapshot:

```http
PUT _snapshot/nba-import-snapshots/<your snapshot name>
{
  "indices": "specimen,taxon,multimedia,geoareas"
}
```

Check the progress of the snapshot with:

```http
GET _snapshot/nba-import-snapshots/<your snapshot name>/_status
```

When the snapshot state is success you can up the number of replica's. First
check the available space via `GET _cat/allocation?v` and calculate the number
of replica's you can take.

For geo configure the number of replicas based on the number of Elasticsearch
nodes minus 1. For example, if your cluster has eight nodes, configure 7
replicas:

```http
PUT /geoareas/_settings
{
  "number_of_replicas": 7
}
```

For `specimen,taxon,multimedia` configure the number of replicas depending on
the available free space, for example:

```http
PUT /specimen,taxon,multimedia/_settings
{
  "number_of_replicas": 2
}
```

Check status of replica's via `GET _cat/indices` and `GET _cat/shards`.

### Undo jobs

In the folder `99-nullify-jobs` there are some jobs to undo a certain import.
