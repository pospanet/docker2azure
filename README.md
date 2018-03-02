# How to run docker image on Azure as fully managed service

There is numbers of options how you can host docker containers on Azure. I will focus only on fully managed services. Some of them are still in preview.

[TOC]



## Working Environment

To keep it as simple as possible I will be using multi-platform tools to cover as much scenarios as possible.

We will need:
- [Azure CLI](#AzureCLI)
- [Docker](#Docker)
- Azure Subscription
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

### Azure subscription

Same situation. If are reading this article, most probably you do have some Azure subscription. If not go [here](https://azure.microsoft.com/), please.

- [Azure free trial](https://azure.microsoft.com/free/)
- [Buy Azure](https://azure.microsoft.com/pricing/purchase-options/)
- [Azure Pass](https://www.microsoftazurepass.com/)
- [Visual Studio Essential](https://www.visualstudio.com/dev-essentials/) with Azure

## How to create Azure Container Registry

If you also need to have private storage for your Docker containers, there is a service for that named  [Azure Container Registry](https://azure.microsoft.com/services/container-registry/). So firstly let's create out private Docker registry.

## Azure Container Registry



- ## Deploying to Azure Container Instance

- ## Deploying to Azure Web App for Containers

- ## Deploying to Azure Container Service

