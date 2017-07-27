# NBA tasks

#### Update API version
```shell
kubectl edit deployment nba-api
```

#### Update Elasticsearch deployment
```shell
kubectl delete deployment elasticsearch
vim elasticsearch/es-deployment.yaml
kubectl create -f elasticsearch/es-deployment.yaml
```

#### Change (log) setting API
```shell
kubectl edit configmap <what you want to change>
kubectl delete deployment nba-api
kubectl create -f api/nba-deployment.yaml
```

#### Update dwca config API
```shell
kubectl delete -f api/dwca-clone-job.yaml
kubectl create -f api/dwca-clone-job.yaml
```

#### Build a new version of API / ETL
```shell
git clone https://github.com/AtzeDeVries/travisci-nba-(api/etl)-docker
git branch
```
Check if the branch (from github.com/naturalis/naturalis_data_api) is 
available, if not; create and push.
Any change in this repo will trigger a build. If there just need to be a 
new build, go to travis-ci.org and find the repo with branch and click
the rebuild button. 
Check on https://hub.docker.com/r/atzedevries/ if there is a new build
available. 
Model of the docker images are [API](https://github.com/AtzeDeVries/docker-nba-api)
[ETL](https://github.com/AtzeDeVries/docker-nba-etl)

#### Build a new version of PURL
```shell
git clone https://github.com/AtzeDeVries/docker-nba-purl
```
Add new `purl.war` and then push to GH. Travis will build a new docker image.

#### Updating source datasets 
There are many sources of data in the NBA. Here is a github list with some extra info
Allmost all source data repositories have two scrips, and a 3 support files 
`compress.sh, uncompress.sh, filelist, .gitignore, README.md`. Compressing works only 
on linux. 

##### [Brahms](https://github.com/naturalis/nba-brondata-brahms)
The dump is a manual (and long process). Done by IM. 

##### [CRS](https://github.com/naturalis/nba-brondata-crs)
The dump is a semi-manual (and long process). Done by IM. 

##### [COL](https://github.com/naturalis/nba-brondata-col)
Only dumps of the static sets. Done by SD. When importing check in 
`nba.properites` if the settings `col-year` matches the dump.

##### [NSR](https://github.com/naturalis/nba-brondata-nsr)
Full dump,compress and push of NSR, is fully automatich including tags

##### [Geo](https://github.com/naturalis/nba-brondata-geo)
Dataset is created and pushed by SD

##### [Medialib](https://github.com/naturalis/nba-brondata-medialib)
This is a supporting dataset, run in on the medialib-dataserver. 
```shell
docker run --rm -it --env-file env.file -v $(pwd)/out:/payload  atzedevries/docker-nba-ml-mimetype
```
And commit it to this repository. Done by Infra

##### [Bijzondere Collectie](https://github.com/naturalis/nba-brondata-bijzcol)
This is a supporting dataset, should be there before a import. Data is injected in etl docker image. 
Data is managed by IM. 








