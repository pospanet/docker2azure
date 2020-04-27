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
az ad sp create-for-rbac --name <Service principal display name /optional/> --role "Contributor" --scopes="/subscriptions/<subscription id>/resourceGroups/<resource group name>"
```

Btw, take a closer look on output from previous command. Can you see. :-) Yes, you can take the scope from there. So did you write it down manually or copy-and-paste?

You should get response like:

![Service Principal][service_principal]

#### Azure SP log-in

Now it is time to login using SP

```
az login --service-principal -u <appId> -p <password> -t <Directory ID>
```

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

Outpus should looks like this:

```json
{
  "additionalProperties": {},
  "containers": [
    {
      "additionalProperties": {},
      "command": null,
      "environmentVariables": [],
      "image": "pospascontainerregistry.azurecr.io/nginx",
      "instanceView": null,
      "name": "pospasinstance",
      "ports": [
        {
          "additionalProperties": {},
          "port": 80,
          "protocol": null
        }
      ],
      "resources": {
        "additionalProperties": {},
        "limits": null,
        "requests": {
          "additionalProperties": {},
          "cpu": 1.0,
          "memoryInGb": 1.0
        }
      },
      "volumeMounts": null
    }
  ],
  "id": "/subscriptions/58f7e072-a001-45c1-a786-26e8da50e91a/resourceGroups/DockerOnAzure/providers/Microsoft.ContainerInstance/containerGroups/pospasinstance",
  "imageRegistryCredentials": [
    {
      "additionalProperties": {},
      "password": null,
      "server": "pospascontainerregistry.azurecr.io",
      "username": "pospascontainerregistry"
    }
  ],
  "instanceView": {
    "additionalProperties": {},
    "events": [],
    "state": "Pending"
  },
  "ipAddress": {
    "additionalProperties": {},
    "dnsNameLabel": "pospasinstance",
    "fqdn": "pospasinstance.westeurope.azurecontainer.io",
    "ip": "13.80.153.169",
    "ports": [
      {
        "additionalProperties": {},
        "port": 80,
        "protocol": "TCP"
      }
    ]
  },
  "location": "westeurope",
  "name": "pospasinstance",
  "osType": "Linux",
  "provisioningState": "Creating",
  "resourceGroup": "DockerOnAzure",
  "restartPolicy": "Always",
  "tags": null,
  "type": "Microsoft.ContainerInstance/containerGroups",
  "volumes": null
}
```

Find parameter *fqdn* or *IP* under *ipAddress* and open it. Of course in case there is a web app inside the container. :-)

> **If you get error message like Subscription is not registered for Microsoft.ContainerInstance namespace, please run**
>
> ```
> az provider register -n Microsoft.ContainerInstance
> ```
> This command must be be running not under SP we have created, but ideally under subscription owner. Best way how to do that is using build-in CLI in Azure Portal or re-login using interactive log-in

![Azure CLU under Azure Portal][azure_portal_cli]

## Deploying to Azure Web App for Containers

Now let's try to do the same for Web App.

### Linux App Service plan

We have to start with the App Service Plan (ASP) to have hosting environment for out app.

```
az appservice plan create -n <ASP name> -g <resource group> --sku <sku> --is-linux
```

SKUs can be found [here](https://docs.microsoft.com/en-us/cli/azure/appservice/plan?view=azure-cli-latest#az_appservice_plan_create).

Next Web App

```
az webapp create -g <resource group name> -p <ASP name> -n <app name> -r <anything :-)>
```

And as a last step, we need to configure Web App to tun our Docker  container

```
az webapp config container set -n <app name> -g <resource group name> -i <ACR name>.azurecr.io/<image name> -r https://<ACR name>.azurecr.io -u <ACR name> -p <ACR password>
```

## Deploying to Azure Container Service (AKS)

Firstly, AKS is not a typo. Acure Container Service abbreviation is really AKS. ACS stands for Access Control Service

### Create AKS

We will use the same *az* command for that

```
az aks create -g <resource group name> -n <cluster name> -p <DNS prefix> -k <Kubernetes version> -s <vm size> -c <node count> -u <ssh user name> --service-principal <service principal> --client-secret <client secret> --generate-ssh-keys
```

full reference can be found [here](https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-create).

To get a full VM size list run

```
az vm list-sizes
```

>**If you get error message like Subscription is not registered for Microsoft.ContainerService namespace, please run**
>
>```
>az provider register -n Microsoft.ContainerService
>```
>
>This command must be be running not under SP we have created, but ideally under subscription owner. Best way how to do that is using build-in CLI in Azure Portal or re-login using interactive log-in

Once done, we have to get credentials out of the cluster. 

```
az aks get-credentials -g <resource group name> -n <cluster name>
```

We will also need to have Kubernetes CLI installed.

For Windows OS run Command Line as Administrator (see below) and then run 

```
az aks install-cli
```

![How to run command line as Administrator][cmd_as_admin]

For Linux based OS run

```
sudo az aks install-cli
```

### Define desired infrastructure

//ToDo


[login_request_cli]: https://github.com/pospanet/docker2azure/blob/master/screenshots/az_login_request.png	"az login request"
[login_request_web]: https://github.com/pospanet/docker2azure/blob/master/screenshots/az_login_web_request.png	"az login web request"
[login_request_web_2]: https://github.com/pospanet/docker2azure/blob/master/screenshots/az_login_web_request_2.png	"az login web request"
[login_response_web]: https://github.com/pospanet/docker2azure/blob/master/screenshots/az_login_web_response.png	"az login web response"
[subscription_list]: https://github.com/pospanet/docker2azure/blob/master/screenshots/az_login_response.png	"account list"
[service_principal]: https://github.com/pospanet/docker2azure/blob/master/screenshots/sp.png	"Service Principal"
[azure_portal]: https://github.com/pospanet/docker2azure/blob/master/screenshots/TenantID.png	"Azure portal"
[acr_ui]: https://github.com/pospanet/docker2azure/blob/master/screenshots/acr_portal.png	"Azure portal - ACR"
[acr_instance]: https://github.com/pospanet/docker2azure/blob/master/screenshots/aci_run_instance.png	"Azure portal - Create Container Instance"
[azure_portal_cli]: https://github.com/pospanet/docker2azure/blob/master/screenshots/azure_portal_cli.png	"Azure portal - CLI"
[cmd_as_admin]: https://github.com/pospanet/docker2azure/blob/master/screenshots/cmd_as_admin.png "Run command line as Administrator"
