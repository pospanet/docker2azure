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

![Login request CLI](login_request_cli)

Please open https://aka.ms/devicelogin and finish authentication.

//ToDo

Maybe you are thinking about some way of automation so interactive login is not an option for you. To do that we will be using service principal next time. To create one we have to chose resource group.

You should get list of subscription after successful login or you can list them by running

```
az account list
```

//ToDo

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

For service principal (SP) we need to specify a scope. In our case it will be newly created resource group. For that we need to find Azure subscription id

//ToDo

Then create SP by running

```
az ad sp create-for-rbac --role "Contributor" --scopes="/subscriptions/<subscription id>/resourceGroups/<resource group name>"
```

Btw, take a closer look on output from previous command. Can you see. :-) Yes, you can take the scope from there. So did you write it down manually or copy-and-paste?

#### Azure SP log-in

Now it is time to login using SP



## How to create Azure Container Registry

If you also need to have private storage for your Docker containers, there is a service for that named  [Azure Container Registry](https://azure.microsoft.com/services/container-registry/). So firstly let's create out private Docker registry.

## Azure Container Registry



- ## Deploying to Azure Container Instance

- ## Deploying to Azure Web App for Containers

- ## Deploying to Azure Container Service


[login_request_cli]: /pospanet/docker2azure/blob/master/screenshots/az_login_request.png	"az login request"