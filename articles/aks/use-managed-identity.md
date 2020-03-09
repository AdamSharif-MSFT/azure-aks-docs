---
title: Use managed identities in Azure Kubernetes Service
description: Learn how to use managed identities in Azure Kubernetes Service (AKS) 
services: container-service
author: saudas
manager: saudas
ms.topic: article
ms.date: 03/10/2019
ms.author: saudas
---

# Use managed identities in Azure Kubernetes Service

Currently, an Azure Kubernetes Service (AKS) cluster (specifically, the Kubernetes cloud provider) requires a *service principal* to create additional resources like load balancers and managed disks in Azure. Either you must provide a service principal or AKS creates one on your behalf. Service principals typically have an expiration date. Clusters eventually reach a state in which the service principal must be renewed to keep the cluster working. Managing service principals adds complexity.

*Managed identities* are essentially a wrapper around service principals, and make their management simpler. To learn more, read about [managed identities for Azure resources](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

AKS creates two managed identities:

- **System-assigned managed identity**: The identity that the Kubernetes cloud provider uses to create Azure resources on behalf of the user. The life cycle of the system-assigned identity is tied to that of the cluster. The identity is deleted when the cluster is deleted.
- **User-assigned managed identity**: The identity that's used for authorization in the cluster. For example, the user-assigned identity is used to authorize AKS to use access control records (ACRs), or to authorize the kubelet to get metadata from Azure.

Any add-ons also authenticate using a managed identity created by the service.

## Before you begin

You must have the following resource installed:

- The Azure CLI, version 2.2.0 or later

## Create an AKS cluster with managed identities

You can now create an AKS cluster with managed identities by using the following CLI commands.

First, create an Azure resource group:

```azurecli-interactive
# Create an Azure resource group
az group create --name myResourceGroup --location westus2
```

Then, create an AKS cluster:

```azurecli-interactive
az aks create -g MyResourceGroup -n MyManagedCluster --enable-managed-identity
```

Finally, get credentials to access the cluster:

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name MyManagedCluster
```

The cluster will be created in a few minutes. You can then deploy your application workloads to the new cluster and interact with it just as you've done with service-principal-based AKS clusters.

> [!IMPORTANT]
>
> - AKS clusters with managed identities can be enabled only during creation of the cluster.
> - Existing AKS clusters cannot be updated or upgraded to enable managed identities.
