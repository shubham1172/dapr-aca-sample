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
docker buildx build --platform linux/amd64 -t daprms.azurecr.io/public/daprio/samples/azcli-capps/python-order-publisher:latest --push ./order-publisher
docker buildx build --platform linux/amd64 -t daprms.azurecr.io/public/daprio/samples/azcli-capps/python-order-processor:latest --push ./order-processor
```

## Deploying container app environment and apps

Do this via portal for now, there is some issue with these commands.

```bash
VAR_RESOURCE_GROUP="dapraca-$(uuidgen | cut -c1-4)"
VAR_ENVIRONMENT="myacaenv"
VAR_LOCATION="eastus"
echo $VAR_RESOURCE_GROUP

## Create the resource group
az group create \
  --name "$VAR_RESOURCE_GROUP" \
  --location "$VAR_LOCATION"

## Create the managed environment
az deployment group create \
  --resource-group "$VAR_RESOURCE_GROUP" \
  --template-file ./deploy/managedEnvironment.bicep \
  --parameters environment_name="$VAR_ENVIRONMENT" \
  --parameters location="$VAR_LOCATION"

## Initialize Dapr components
az containerapp env dapr-component init -g $VAR_RESOURCE_GROUP --name $VAR_ENVIRONMENT 

## Deploy the container apps
az deployment group create \
  --resource-group "$VAR_RESOURCE_GROUP" \
  --template-file ./deploy/containerApps.bicep \
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