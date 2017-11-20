# Requirements for deployeing NBA v2

### Kubernetes
A kubernetes (version > 1.7) is required for the NBA. A 1.6 cluster is also possible but
some changes are needed in the deployment steps.

### Kubernetes resources

#### Ingress
A ingress controller is needed. Deployment is tested with `traefik` but other ingress controllers
should not be a problem

#### Storageclass

##### Blockdevices
A RWO block devices is needed for Minio and A NFS storage server. In the current setup a storage class
`cinder` is deployed. 

##### Shared storage
A RWX storage class is needed for shared storage. A nfs providers (a default example) is needed for this.

#### Persistant volumes
For elasticsearch to run as a statefull set, persistend localdisk volumes support is required. Also the folder
(/es-data) is needed to be created on all minions. 




