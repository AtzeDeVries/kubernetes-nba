# NBA tasks

#### Update API version
```shell
kubectl set image deployment/nba-api nba-api=<new image name> (for ex naturalis/nba-api:V2.11.3-e39fae)
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

#### Updating source datasets 
There are many sources of data in the NBA. Here is a github list with some extra info
Allmost all source data repositories have two scrips, and a 3 support files 
`compress.sh, uncompress.sh, filelist, .gitignore, README.md`. Compressing works only 
on linux. 

##### [Brahms](https://github.com/naturalis/nba-brondata-brahms)
The dump is a manual (and long process). Done by IM. 

##### [CRS](https://github.com/naturalis/nba-brondata-crs)
CRS is moved to drive, for speed improvements. A better how to for this is
needed.

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








