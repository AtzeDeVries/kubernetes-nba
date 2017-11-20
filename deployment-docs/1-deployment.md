# Deployment of NBA v2

### Secrets
First create the secrets from the secrets-template directory
```SHELL
kubectl create -f -R  sercrets/
```

### Elasticsearch
```SHELL
vim elasticsearch/es-data-pv.yaml # check if hostnames are matching
kubectl create -f elasticsearch/es-discovery-svc.yaml
kubectl create -f elasticsearch/es-data-pv.yaml
kubectl get pv
kubectl create -f elasticsearch/es-data-stateful.yaml
kubectl create -f elasticsearch/es-data-svc.yaml
kubectl get pods -w
```

### Fanout
```SHELL
kubectl create -f api-ingress/
```

### Api
```SHELL
kubectl create -f api/
kubectl get pods -w
kubectl scale deployment/nba-api --replicas=2
```

### Purl
```SHELL
kubectl create -f purl/
kubectl get pods -w
kubectl scale deployment/nba-purl --replicas=2
```

### Scratchpad
```SHELL
kubectl create -f scratchpad/
kubectl get pods -w
kubectl scale deployment/nba-scratchpad --replicas=2
```

### Direct query
```SHELL
kubectl create -f nba-direct-api/
kubectl get pods -w
```

### Docs
```SHELL
kubectl create -f docs/
kubectl get pods -w
kubectl scale deployment/nba-docs --replicas=2
kubectl scale deployment/nba-swagger --replicas=2
```

### Entrypoint
```SHELL
kubectl create -f nba-entrypoint/
kubectl get pods -w
kubectl scale deployment/nba-entrypoint--replicas=2
```

### Docs redirect
```SHELL
kubectl create -f docs-redirect/
kubectl get pods -w
kubectl scale deployment/nba-docs-redirect --replicas=2
```

### Kibana (not needed)
```SHELL
kubectl create -f kibana/
kubectl get svc for kibanan port nr
```



