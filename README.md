# How to run docker image on Azure as fully managed service

There is numbers of options how you can host docker containers on Azure. I will focus only on fully managed services. Some of them are still in preview.

[TOC]



## Working Environment

To keep it as simple as possible I will be using multi-platform tools to cover as much scenarios as possible.

We will need:
- [Azure CLI](#AzureCLI)
- [Docker](#Docker)
- [Azure Subscription](#Azure)
### Azure CLI <a id="AzureCLI"/>

[Azure CLI](https://docs.microsoft.com/cli/azure/) is a multi-platform tool for managing Azure resources. You can download and install from [here](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).

If you are installing Azure CLI into docker container, use this command.

```python
pip install azure-cli
```

### Docker <a id="Docker" />

You are just about to deploy Docker container, so you should have Docker installed, right?. If not download from [here](https://www.docker.com/community-edition#/download).

- [Docker CE for Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows)
- [Docker CE for Mac](https://store.docker.com/editions/community/docker-ce-desktop-mac)
- [Docker CE for Ubuntu](https://store.docker.com/editions/community/docker-ce-server-ubuntu)

### Azure subscription <a id="Azure" />

Same situation. If are reading this article, most probably you do have some Azure subscription. If not go [here](https://azure.microsoft.com/), please.

- [Azure free trial](https://azure.microsoft.com/free/)
- [Buy Azure](https://azure.microsoft.com/pricing/purchase-options/)
- [Azure Pass](https://www.microsoftazurepass.com/)
- [Visual Studio Essential](https://www.visualstudio.com/dev-essentials/) with Azure

## Setting-up environment

#### Azure Interactive log-in

Once all tools are install, it is time to setup Azure subscription. Open Azure CLI and run

```
az login
```

You will get request for interactive login like:

![Login request CLI][login_request_cli]

Please open https://aka.ms/devicelogin and finish authentication.

![Login request web][login_request_web]

![Login request web ][login_request_web_2]

![Login response web][login_response_web]

Maybe you are thinking about some way of automation so interactive login is not an option for you. To do that we will be using service principal next time. To create one we have to chose resource group.

You should get list of subscription after successful login or you can list them by running

```
az account list
```

![account list][subscription_list]

Find you subscription and select it by running

```
az account set -s <subscription name>
```

Write down even Subscription ID for later use.

#### Create Resource group

I'd like to simplify command parameters so we can set default location for our resources. Doing them we can let out future *--location*

```
az configure --defaults location=<location>
```

You can list all locations available for selected subscription by running

```
az account list-locations
```

Then create resource group

```
az group create -n <resource group name>
```

#### Create Service Principal

For service principal (SP) we need to specify a scope. In our case it will be newly created resource group.

Then create SP by running

```
az ad sp create-for-rbac --role "Contributor" --scopes="/subscriptions/<subscription id>/resourceGroups/<resource group name>"
```

Btw, take a closer look on output from previous command. Can you see. :-) Yes, you can take the scope from there. So did you write it down manually or copy-and-paste?

You should get response like:

![Service Principal][service_principal]

#### Azure SP log-in

Now it is time to login using SP

```
az login --service-principal -u <appId> -p <password> -t <Directory ID>
```

Directory ID aka Tenant ID can be found here

![Azure Portal][azure_portal]

## How to create Azure Container Registry

If you also need to have private storage for your Docker containers, there is a service for that named  [Azure Container Registry](https://azure.microsoft.com/services/container-registry/) (ACR). So firstly let's create out private Docker registry.

## Azure Container Registry

To create ACR we just need to specify [SKU](http://docs.microsoft.com/azure/container-registry/container-registry-skus) by running

```
az acr create --resource-group <resource group name> --name <container registry name> --sku <sku> --admin-enabled true
```

### Pushing Docker image into ACR

Let's have your own Docker image ready. Then we have to log-in to ACR by running

```
az acr login -n <acr name>
```

Tag image

```
docker tag <image name> <acr name>.azurecr.io/<image name>
```

Next we need to authorize docker to ACR. User name is ACR name and password can be retrieved by running

```
az acr credential show -n <ACR name> -o table
```

Then just simply

```
docker login <ACR name>.azurecr.io -u <ACR name> -p <password>
```

and push the image

```
docker push <ACR name>.azurecr.io/<image name>
```

You can double-check by running

```
az acr repository list -n <ACR name> -o table
```

## Deploying to Azure Container Instance

```
az container create -g <resource group> -n <instance name> --image <ACR name>.azurecr.io/<image name> --cpu <#core> --memory <#GB> --os-type <Linux|Windows> --ports <space separated ports> --registry-password <ACR password> --dns-name-label <dns name label for group with public IP>
```

Full list of parameters [here](https://docs.microsoft.com/en-us/cli/azure/container?view=azure-cli-latest#az_container_create).

## Deploying to Azure Web App for Containers

## Deploying to Azure Container Service

[login_request_cli]: https://github.com/pospanet/docker2azure/blob/master/screenshots/az_login_request.png	"az login request"
[login_request_web]: https://github.com/pospanet/docker2azure/blob/master/screenshots/az_login_web_request.png	"az login web request"
[login_request_web_2]: https://github.com/pospanet/docker2azure/blob/master/screenshots/az_login_web_request_2.png	"az login web request"
[login_response_web]: https://github.com/pospanet/docker2azure/blob/master/screenshots/az_login_web_response.png	"az login web response"
[subscription_list]: https://github.com/pospanet/docker2azure/blob/master/screenshots/az_login_response.png	"account list"
[service_principal]: https://github.com/pospanet/docker2azure/blob/master/screenshots/sp.png	"Service Principal"
[azure_portal]: https://github.com/pospanet/docker2azure/blob/master/screenshots/TenantID.png	"Azure portal"