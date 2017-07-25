# NBA tasks

#### Update NBA version
```shell
kubectl edit deployment nba-api
```

#### Update Elasticsearch deployment
```shell
kubectl delete deployment elasticsearch
vim elasticsearch/es-deployment.yaml
kubectl create -f elasticsearch/es-deployment.yaml
```

#### Change (log) setting NBA
```shell
kubectl edit configmap <what you want to change>
kubectl delete deployment nba-api
kubectl create -f api/nba-deployment.yaml
```

#### Update dwca config NBA
```shell
kubectl delete -f nba/dwca-clone-job.yaml
kubectl create -f nba/dwca-clone-job.yaml
```
