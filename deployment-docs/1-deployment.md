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

### FANOUT
```SHELL
kubectl create -f api-ingress/
```

### API
```SHELL
kubectl create -f api/
kubectl get pods -w
kubectl scale deployment/nba-api --replicas=2
```

### PURL
```SHELL
kubectl create -f purl/
kubectl get pods -w
kubectl scale deployment/nba-purl --replicas=2
```

### SCRATCHPAD
```SHELL
kubectl create -f scratchpad/
kubectl get pods -w
kubectl scale deployment/nba-scratchpad --replicas=2
```

### DIRECT QUERY
```SHELL
kubectl create -f nba-direct-api/
kubectl get pods -w
```

### DOCS
```SHELL
kubectl create -f docs/
kubectl get pods -w
kubectl scale deployment/nba-docs --replicas=2
kubectl scale deployment/nba-swagger --replicas=2
```

### ENTRYPOINT
```SHELL
kubectl create -f nba-entrypoint/
kubectl get pods -w
kubectl scale deployment/nba-entrypoint--replicas=2
```

### DOCS REDIRECT
```SHELL
kubectl create -f docs-redirect/
kubectl get pods -w
kubectl scale deployment/nba-docs-redirect --replicas=2
```



