---
title: Integrate with Azure-managed services using Open Service Broker for Azure (OSBA) | Microsoft Docs
description: Integrate with Azure-managed services using Open Service Broker for Azure (OSBA)
services: container-service
documentationcenter: ''
author: sozercan
manager: ritazh
editor: ''
tags: aks, azure-container-service
keywords: Kubernetes, Docker, Containers, Microservices, Azure, Open Service Broker

ms.assetid:
ms.service: container-service
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/30/2017
ms.author: seozerca
ms.custom: mvc

---
# Integrate with Azure-managed services using Open Service Broker for Azure (OSBA)

Together with the [Kubernetes Service Catalog](https://github.com/kubernetes-incubator/service-catalog), Open Service Broker for Azure (OSBA) allows developers to utilize Azure-managed services in Kubernetes. This guide focuses on deploying Kubernetes Service Catalog, Open Service Broker for Azure (OSBA), and applications that use Azure-managed services using Kubernetes.

## Prerequisites
* An Azure subscription

* Azure CLI 2.0: You can [install it locally](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) or use it in the [Azure Cloud Shell](https://docs.microsoft.com/en-us/azure/cloud-shell/overview)

* Helm CLI 2.7+: You can [install it locally](https://docs.microsoft.com/en-us/azure/aks/kubernetes-helm#install-helm-cli) or use it in the [Azure Cloud Shell](https://docs.microsoft.com/en-us/azure/cloud-shell/overview)

* Permissions to create a service principal with the Contributor role on your Azure subscription

* Creating an Azure Container Service (AKS) cluster

Let's get started by deploying an Azure Container Service (AKS) cluster. You can follow the [Create an AKS cluster](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough) quickstart guide.

## Installing Service Catalog

Next step is to install service catalog using Helm chart. Refer to [Install Helm CLI](https://docs.microsoft.com/en-us/azure/aks/kubernetes-helm#install-helm-cli) for Helm installation.
Download and install Helm CLI 2.7.2+ in your local environment, and upgrade your Tiller (Helm server) installation with:

```azurecli-interactive
helm init --upgrade
```

Adding the service catalog chart Helm repository:

```azurecli-interactive
helm repo add svc-cat https://svc-catalog-charts.storage.googleapis.com
```

Installing service catalog from Helm chart:

```azurecli-interactive
helm install svc-cat/catalog \
    --name catalog --namespace catalog --set rbacEnable=false
```

After, you install the Helm chart, you should see service catalog in:

```azurecli-interactive
kubectl get apiservice
```

## Installing Open Service Broker for Azure

Next step is to install [Open Service Broker for Azure](https://github.com/Azure/open-service-broker-azure), which includes the catalog for the Azure-managed services. Examples for available Azure services are Azure Database for PostgreSQL, Azure Redis Cache, Azure Database for MySQL, Azure Cosmos DB, Azure SQL Database, and others. 

Let's start by adding Open Service Broker for Azure Helm repository:

```azurecli-interactive
helm repo add azure https://kubernetescharts.blob.core.windows.net/azure
```

Create a [Service Principal](https://docs.microsoft.com/en-us/azure/aks/kubernetes-service-principal) using:

```azurecli-interactive
az ad sp create-for-rbac --skip-assignment
```

Output looks something like the example below, `appId`, `password` and `tenant` values are used in the next step.

```
{
  "appId": "7248f250-0000-0000-0000-dbdeb8400d85",
  "displayName": "azure-cli-2017-10-15-02-20-15",
  "name": "http://azure-cli-2017-10-15-02-20-15",
  "password": "77851d2c-0000-0000-0000-cb3ebc97975a",
  "tenant": "72f988bf-0000-0000-0000-2d7cd011db47"
}
```

Setting up environment variables with the preceding values:

```azurecli-interactive
AZURE_CLIENT_ID=[your Azure appId from above]
AZURE_CLIENT_SECRET=[your Azure password from above]
AZURE_TENANT_ID=[your Azure tenant from above]
```

and get subscription using
```azurecli-interactive
az account show
```

Output looks something like the example below, `id` value is used for subscription ID.

```
{
  "environmentName": "AzureCloud",
  "id": "33ee811e-12bf-4g5c-9c62-a2f28c5f7724",
  "isDefault": true,
  "name": "Azure Subscription,
  "state": "Enabled",
  "tenantId": "72f988bf-0000-0000-0000-2d7cd011db47",
  "user": {
    "name": "user@example.com",
    "type": "user"
  }
}
```

Setting up environment variable with the preceding value:

```azurecli-interactive
AZURE_SUBSCRIPTION_ID=[your Azure subscription Id from above]
```

Installing Open Service Broker for Azure using Helm chart:

```azurecli-interactive
helm install azure/open-service-broker-azure --name osba --namespace osba \
    --set azure.subscriptionId=$AZURE_SUBSCRIPTION_ID \
    --set azure.tenantId=$AZURE_TENANT_ID \
    --set azure.clientId=$AZURE_CLIENT_ID \
    --set azure.clientSecret=$AZURE_CLIENT_SECRET
```

Here are a few commands to verify Open Service Broker for Azure and installed services with plans:

Show installed broker:
```azurecli-interactive
kubectl get clusterservicebroker -o yaml
```

Show installed service classes:
```azurecli-interactive
kubectl get clusterserviceclasses -o=custom-columns=NAME:.metadata.name,EXTERNALNAME:.spec.externalName
```

Show installed service plans:
```azurecli-interactive
kubectl get clusterserviceplans \
    -o=custom-columns=NAME:.metadata.name,EXTERNALNAME:.spec.externalName,SERVICECLASS:.spec.clusterServiceClassRef.name \
    --sort-by=.spec.clusterServiceClassRef.name
```

## Installing WordPress from Helm chart using Azure Database for MySQL

This command uses Helm to install an updated Helm chart of WordPress that provisions an external Azure Database for MySQL instance in order to have WordPress use it. This process can take a few minutes.

```azurecli-interactive
helm install azure/wordpress --name wordpress --namespace wordpress
```

In order to verify the installation has provisioned the right resources:

Show installed service instance and bindings:

```azurecli-interactive
kubectl get serviceinstance -n wordpress -o yaml
kubectl get servicebinding -n wordpress -o yaml
```

Show installed secrets:

```azurecli-interactive
kubectl get secrets -n wordpress -o yaml
```

If you are interested in more charts to deploy using Open Service Broker for Azure, check out [Azure/helm-charts](https://github.com/Azure/helm-charts) repository.

## Summary

This guide showed how to install Service Catalog to an Azure Container Service (AKS) cluster, how to deploy Open Service Broker for Azure and deploy an application, such as WordPress, that is using Azure-managed services, in this case Azure Database for MySQL.
