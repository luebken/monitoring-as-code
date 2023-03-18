# Monitoring-As-Code with Crossplane

## Setup

Let's create a local Kind cluster with Crossplane installed:
```sh
kind create cluster --image kindest/node:v1.26.2
helm install crossplane --namespace crossplane-system --create-namespace crossplane-stable/crossplane
# verify
kubectl get pods -n crossplane-system
```

## Grafana
The first example is with Grafana Cloud. 

For this we need to talk to two APIs: the cloud api to create a stack and afterwards to the stacks instance api to create the dashboards.

See also https://grafana.com/docs/grafana-cloud/infrastructure-as-code/terraform/terraform-cloud-stack and https://github.com/grafana/crossplane-provider-grafana

```sh
# install provider
kubectl crossplane install provider xpkg.upbound.io/grafana/provider-grafana:v0.3.0

# Create a cloud api key here: https://grafana.com/orgs/<your-org>/api-keys
# and insert the key into 1.grafana/grafana-cloud-secret.yaml
kubectl apply -f grafana/1.grafana-cloud-secret.yaml
kubectl apply -f grafana/2.grafana-cloud-provider-config.yaml
# choose a unique slug name
# and insert into 3.stack.yaml
kubectl apply -f grafana/3.stack.yaml
# verify:
kubectl describe stack.cloud.grafana.crossplane.io/my-stack
open https://<slug>.grafana.net/

# Create an API for the instance:
kubectl apply -f grafana/4.api-key-from-stack.yaml
# Configure provider with the generated secret:
kubectl apply -f grafana/5.grafana-instance-provider-config.yaml
# Create dashboard
kubectl apply -f grafana/6.dashboard.yaml

kubectl get dashboard.oss.grafana.crossplane.io/cloud-stack-dashboard -o yaml | yq .status.atProvider.url
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