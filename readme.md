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
export GRAFANA_CLOUD_API_KEY=...
curl -X POST --insecure -H "Authorization: Bearer ${GRAFANA_CLOUD_API_KEY}" -H "Content-Type: application/json" -d '{
  "dashboard": {
    "title": "Test Dashboard"
  }
}' https://luebken.grafana.net/api/dashboards/db
```

## Datadog
https://github.com/crossplane-contrib/provider-jet-datadog/

https://app.datadoghq.eu/organization-settings/api-keys
https://app.datadoghq.eu/organization-settings/application-keys

Curl
```
export DD_API_KEY=...
export DD_APP_KEY=...
curl -X POST "https://api.datadoghq.eu/api/v1/dashboard" \
-H "Accept: application/json" \
-H "Content-Type: application/json" \
-H "DD-API-KEY: ${DD_API_KEY}" \
-H "DD-APPLICATION-KEY: ${DD_APP_KEY}" \
-d @- << EOF
{
  "title": "Crossplane Test",
  "widgets": [
    {
      "definition": {
        "title": "Crossplane Test",
        "type": "distribution",
        "requests": [
          {
            "query": {
              "stat": "latency_distribution",
              "data_source": "apm_resource_stats",
              "name": "query1",
              "service": "azure-bill-import",
              "group_by": [
                "resource_name"
              ],
              "env": "staging",
              "operation_name": "universal.http.client"
            },
            "request_type": "histogram"
          }
        ]
      }
    }
  ],
  "layout_type": "ordered"
}
EOF
```