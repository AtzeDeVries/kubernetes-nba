# Workflow for a NBA deployment


### Requirements
* A recent (1.6.0 or higher) kubernetes cluster 
* kubectl client
* this repo
* a block based storage class (currently using Openstack Cinder)

### Overview
This deployment will setup a empty NBA cluster with all
services. Importing of the dataset will be in a different document.
The following componets will be deployed
* Elasticsearch
* Logging volumes
* NBA Api
* NBA Purl
* (Kibana)
For import look at `nba-import.md`

#### Elasticsearch
Check the `elasticsearch/es-deployment.yaml` for 3 things. The number of 
replica's should match your cluster. Also the `minimal master` nodes should
be `(number of master-eligible nodes / 2) + 1`. Also very import the `XMS` 
and `XMX` values should be set. This value should be the half of your instances 
RAM.

To deploy
`kubectl create -f elasticsearch/`

#### Logging volumes
This will create a few things. First shared storage server based on NFS with
a storage class. Also persistenVolumeClaims will be made for api, purl and etl logging. 
Minio can be deployed in order to expose the logfile through http. 
Check `logserver-deployment.yaml` to check if the size of the logvolume is sufficient

First create the nfs server and storage class
```
kubectl create -f logserver-deployment.yaml
kubectl create -f logserver-storageclaim.yaml
```

Then make the claims
```
kubectl create -f etl-nfspvc.yaml
kubectl create -f apipurl-nfs-pvc.yaml
```
To expose the logs via http, first check `minio-deploy.yaml` to change stuff
like the password etc. Then
```
kubectl create -f minio-deploy.yaml
kubectl create -f minio-svc.yaml
```
#### API
Check the API version in `nba-deployment.yaml`, also check the dwca git tag in 
`dwca-clone-job.yaml`. Then
`kubectl create -f api/

#### PURL
Check the version in `purl-deployment.yaml` Then `kubectl apply -f purl/` 


#### Kibana
This is not required for a production. But is handy in import/development settings
```shell
kubectl create -f kibana/
```
