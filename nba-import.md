# NBA Import

### Overview
There are two ways to put data in a elasticsearch cluster. Restore using
a snapshot or run a import. First we will discuss the snapshot method, then
the import method.
The current import requires about 500GB of storage. 

## Snapshot
Requirements
* A snapshot (S3 perfered) repository with a snapshot
* Access to kibana or http access of Elasticsearch

#### Config Repository
The manual is based on kibana. Modifying to curl is easy. First check if
there already is a repository
```
GET _snapshot/_all
```
if not add a repository
```
PUT _snapshot/nba-import-snapshots
{
  "type": "s3",
  "settings": {
    "bucket" : "nba-import-snapshots", 
    "region" : "us-east-1",
    "endpoint" : "http://10.233.30.173:9000",
    "access_key" : "changeme",
    "secret_key" : "reallyreally",
    "protocol" : "http"
    "max_retry": "25",
    "max_snapshot_bytes_per_sec": "150mb"
  }
}
```
Check the available snapshots
```
GET _snapshot/nba-import-snapshots/_all
```
#### Run restore
You should either close the to be restored indices or delete them. For NBA 
deleting them is the easy choice since the snapshots are not incremental. Delete the
indices by
```
DELETE /specimen,taxon,multimedia,geoares
```
Then run a restore
```
POST /_snapshot/nba-import-snapshots/<snapshot name>/_restore
{
  "indices" : "specimen,taxon,multimedia,geoares"
}
```
Check status of indices with
```
GET _cat/indices
```

After restore is done, the number of replicas van be changed to the desired state
```
PUT /specmen,multimedia,taxon/_settings
{
  "number_of_replicas": 1
}
```

## Import

To a import go to the folder `import/parajobs`. The import has 4 phases. First check
if there are any jobs this in k8s. 
To delete all run from the current folder
`kubectl delete -R -f ./`

Before a import, check the followig things. 
* import config `0-prepare/etl-configmap-nba.propierties`
* etl versions (docker tag) in the folders 0,1,2 
* source data versions (git tags) in folder 1

#### Prepare indices
`kubectl create -f 0-prepare/`
Wait until the job is done. You can check the job status via
`kubectl get jobs`. Or check pod status via `kubectl get pods --watch`

#### Import sources
`kubectl create -f 1-source-import/`
This process can take between 2 and 5 hours depending on your import cluster.

#### Enrich data
Wait until the import sources is done
`kubectl create -f 2-enrich/`
This will add extra data to all documents. This step takes about 3 hours. 

#### Postjobs
Wait until the enrichtments are done
`kubectl create -f 3-postjobs`. This job will cleanup all deleted documents 
in the indices. This saves space and enchanses query performance. 

#### Undo jobs
In the folder `99-nullify-jobs` there are some jobs to undo a certain import. 



