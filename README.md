## Running locally
```bash
# Start the order processor service
cd order-processor
export DAPR_STATESTORE_COMPONENT=statestore
export DAPR_PUBSUB_COMPONENT=pubsub
dapr run --app-id order-processor --app-port 6001 -- python3 app.py

# In a separate terminal, start the order publisher service
cd order-publisher
export DAPR_PUBSUB_COMPONENT=pubsub
dapr run --app-id order-publisher -- python3 app.py
```

## Running on ACA

### Pre-requisites (optional)

Package the order-processor and order-publisher services into Docker images and push them to a Docker registry. You can use the following commands to build and push the images to your Docker Hub account:

```bash
docker buildx build --platform linux/amd64 -t ghcr.io/shubham1172/aca-dapr-example/order-processor:latest --push ./order-processor
docker buildx build --platform linux/amd64 -t ghcr.io/shubham1172/aca-dapr-example/order-publisher:latest --push ./order-publisher
```

## Deploying the services

Do this via portal for now, there is some issue with these commands.

```bash
# add random suffix to avoid name collisions
VAR_RESOURCE_GROUP="dapraca$(uuidgen | cut -c1-4)"
VAR_ENVIRONMENT="myacaenv"
VAR_LOCATION="eastus"
az deployment group create \
  --name AcaDeployment \
  --resource-group "$VAR_RESOURCE_GROUP" \
  --template-file ./deploy/deploy.bicep \
  --parameters environment_name="$VAR_ENVIRONMENT" \
  --parameters location="$VAR_LOCATION"
```

## View logs

```kql
ContainerAppConsoleLogs_CL
// | summarize min(time_t), max(time_t) by RevisionName_s
| where RevisionName_s in ("order-publisher--fi3vasv", "order-processor--c236r52")
| where ContainerName_s == "daprd"
| project time_t, RevisionName_s, Log_s
| order by time_t asc
```

## Cleanup

```bash
# Delete the resource group
az group delete --name "$VAR_RESOURCE_GROUP"
```