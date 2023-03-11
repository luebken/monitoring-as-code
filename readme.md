# Monitoring-As-Code with Crossplane

## Setup
```
kind create cluster --image kindest/node:v1.26.2
helm install crossplane --namespace crossplane-system --create-namespace crossplane-stable/crossplane
```

## Grafana
https://github.com/grafana/crossplane-provider-grafana
```
kubectl crossplane install provider xpkg.upbound.io/grafana/provider-grafana:v0.3.0
kubectl apply -f grafana/grafana-secret.yaml
kubectl apply -f grafana/provider-config.yaml
kubectl apply -f grafana/dashboard.yaml
```

Curl
```
curl -X POST --insecure -H "Authorization: Bearer KEY" -H "Content-Type: application/json" -d '{
  "dashboard": {
    "title": "Test Dashboard"
  }
}' https://luebken.grafana.net/api/dashboards/db
```



