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

If that's not the case, add a repository (details etc can be changed):

```http
PUT _snapshot/nba-import-snapshots
{
  "type": "s3",
  "settings": {
    "bucket" : "nba-import-snapshots",
    "region" : "us-east-1",
    "endpoint" : "http://ip-addrress:9000",
    "access_key" : "access_key",
    "secret_key" : "secret_key",
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

### Update production

The update of production follows 7 steps
1. Change number of replica's to create space for a restore
2. Restore the new dataset
3. Change aliasses
4. Restart NBA api
5. Verify data
6. Remove old data
7. Update number replica's.

#### Change number of replica's
Note that `-live-gang-okt-2017` changes with new imports
```
PUT specimen-live-gang-okt-2017,taxon-live-gang-okt-2017,multimedia-live-gang-okt-2017,storageunits-live-gang-okt-2017/_settings
{
 "number_of_replicas": 1
}
```
#### Restore a new dataset
Run the restore and rename to indices to a timestamp
```
POST /_snapshot/nba-import-snapshots/nba-import-2017-11-14/_restore
{
  "rename_pattern": "(.+)",
  "rename_replacement": "nba-import-2017-11-14-$1"
}
```
Wait for restore to be ready (can take some hours)

#### Update aliasses
```
POST /_aliases
{
    "actions" : [
        { "add" : { "index" : "nba-import-2017-11-14-specimen", "alias" : "specimen" } },
        { "add" : { "index" : "nba-import-2017-11-14-geoareas", "alias" : "geoareas" } },
        { "add" : { "index" : "nba-import-2017-11-14-multimedia", "alias" : "multimedia" } },
        { "add" : { "index" : "nba-import-2017-11-14-taxon", "alias" : "taxon" } },
        { "add" : { "index" : "nba-import-2017-11-14-storageunits", "alias" : "storageunits" } },
        { "remove" : { "index" : "specimen-live-gang-okt-2017", "alias" : "specimen" } },
        { "remove" : { "index" : "geoareas-live-gang-okt-2017", "alias" : "geoareas" } },
        { "remove" : { "index" : "multimedia-live-gang-okt-2017", "alias" : "multimedia" } },
        { "remove" : { "index" : "taxon-live-gang-okt-2017", "alias" : "taxon" } },
        { "remove" : { "index" : "storageunits-live-gang-okt-2017", "alias" : "storageunits" } }
    ]
}

```
#### Restart NBA
Due to cache the nba api has to be restarted. Kubernetes does not have a restart command. The best way to do this
without downtime is:
```
kubectl get pods -l app=nba-api
```
Then for each nba-api pod do (and wait for a new one to be started after doing a new one)
```
kubectl delete pod <pod name>
```

#### Verify data
Check with some key users that the data is correct

#### Remove old data
```
DELETE /*-live-gang-okt-2017
```
#### Update number of replica's
```
PUT /nba-import-2017-11-14-*/_settings
{
 "number_of_replicas": 2
}
PUT /nba-import-2017-11-14-geoareas/_settings
{
 "number_of_replicas": 7
}
```


## Import

The import is now handeld and fully automated by a jobernetes job. Info can be found here:
https://github.com/AtzeDeVries/jobernetes-etl
