# Monitoring-As-Code with Crossplane

Crossplane manages any type of external API. Traditionally these are cloud infrastructure resources such as AWS S3, Azure CosmosDB or GCP Instance but nothing stops us from creating other type of resources. In this repo we want show on how to create monitoring dashboards.

Why? Because defining concerns like monitoring as K8s yaml lets you bundle these concerns with your application and move these around different environments. It also allows for a clean seperation between defining a dashboard and using a dashboard in any K8s based platform

## First example: Dashboards

### Prerequisite: A K8s Cluster

Let's create a local Kind cluster with Crossplane installed:

```sh
kind create cluster --image kindest/node:v1.26.2
helm install crossplane --namespace crossplane-system --create-namespace crossplane-stable/crossplane
# verify
kubectl get pods -n crossplane-system
```

### Grafana
In the following example we are using the managed Grafana offering [Grafana Cloud](https://grafana.com/products/cloud/) but this works also for self-managed Grafana options. 

For Grafan Cloud we need to talk to two APIs: 
a) the cloud api to create a stack 
and afterwards 
b) the stacks instance api to create the dashboards, alerts, etc.

See also 
* https://grafana.com/docs/grafana-cloud/infrastructure-as-code/terraform/terraform-cloud-stack
* https://github.com/grafana/crossplane-provider-grafana

```sh
# Install the Crossplane provider:
kubectl crossplane install provider xpkg.upbound.io/grafana/provider-grafana:v0.3.0

# a) Cloud API:
# Create a cloud api key here: https://grafana.com/orgs/<your-org>/api-keys and insert "cloud_api_key"
kubectl apply -f grafana/1.grafana-cloud-secret.yaml
kubectl apply -f grafana/2.grafana-cloud-provider-config.yaml
# Choose a unique slug name for you stack:
kubectl apply -f grafana/3.stack.yaml
# verify:
kubectl describe stack.cloud.grafana.crossplane.io/my-stack
open https://<slug>.grafana.net/
```

```sh
# b) Stacks Instance API
# Create an API key for the instance:
kubectl apply -f grafana/4.api-key-from-stack.yaml
# Configure provider with the generated secret:
kubectl apply -f grafana/5.grafana-instance-provider-config.yaml

# Create a blank dashboard:
kubectl apply -f grafana/6.blank-dashboard.yaml
kubectl get dashboard.oss.grafana.crossplane.io/blank-dashboard -o yaml | yq .status.atProvider.url
\o/
```

In our next step let's create dashboards for some common AWS services

```sh
# https://grafana.com/grafana/dashboards/575-aws-s3/

# Create a secret for cloudwatch:
# accessKey: ($ aws configure get aws_access_key_id)
# secretKey: $(aws configure get aws_secret_access_key):
kubectl apply -f grafana/7.aws-cloudwatch-secret.yaml

# Create a AWS Cloudwatch datasource:
# https://registry.terraform.io/providers/grafana/grafana/latest/docs/resources/data_source
# https://marketplace.upbound.io/providers/grafana/provider-grafana/v0.3.0/resources/oss.grafana.crossplane.io/DataSource/v1alpha1
# check under https://<slug>.grafana.net/datasources/
kubectl apply -f grafana/8.aws-cloudwatch-datasource.yaml

# Create an AWS S3 dashboard
# https://grafana.com/grafana/dashboards/575-aws-s3/
kubectl apply -f grafana/9.aws-s3-dashboard.yaml
kubectl get dashboard.oss.grafana.crossplane.io/aws-s3-dashboard -o yaml | yq .status.atProvider.url

\o/
```

### (WIP) Datadog
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