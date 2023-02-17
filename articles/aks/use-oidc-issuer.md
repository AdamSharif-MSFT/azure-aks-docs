---
title: Create an OpenID Connect provider for your Azure Kubernetes Service (AKS) cluster
description: Learn how to configure the OpenID Connect (OIDC) provider for a cluster in Azure Kubernetes Service (AKS)
ms.topic: article
ms.date: 02/16/2023
---

# Create an OpenID Connect provider on Azure Kubernetes Service (AKS)

OpenID Connect (OIDC) extends the OAuth 2.0 authorization protocol for use as an additional authentication protocol. You can use OIDC to enable single sign-on (SSO) between your OAuth-enabled applications by using a security token called an ID token. With your AKS cluster, you can enable OpenID Connect (OIDC) Issuer, which allows Azure Active Directory (Azure AD) or other cloud provider identity and access management platform, to discover the API server's public signing keys.

AKS rotates the key automatically and periodically. If you don't want to wait, you can rotate the key manually and immediately. The maximum lifetime of the token issued by the OIDC provider is one day.

> [!WARNING]
> Enable or disable OIDC Issuer changes the current service account token issuer to a new value, which can cause down time and restarts the API server. If the application pods using a service token remain in a failed state after you enable or disable the OIDC Issuer, we recommend you manually restart the pods.

## Prerequisites

* The Azure CLI version 2.42.0 or higher. Run `az --version` to find your version. If you need to install or upgrade, see [Install Azure CLI][azure-cli-install].
* AKS supports OIDC Issuer on version 1.22 and higher.

## Create an AKS cluster with OIDC Issuer

You can create an AKS cluster using the [az aks create][az-aks-create] command with the `--enable-oidc-issuer` parameter to use the OIDC Issuer. The following example creates a cluster named *myAKSCluster* with one node in the *myResourceGroup*:

```azurecli-interactive
az aks create -g myResourceGroup -n myAKSCluster --node-count 1 --enable-oidc-issuer
```

## Update an AKS cluster with OIDC Issuer

You can update an AKS cluster using the [az aks update][az-aks-update] command with the `--enable-oidc-issuer` parameter to use the OIDC Issuer. The following example updates a cluster named *myAKSCluster*:

```azurecli-interactive
az aks update -g myResourceGroup -n myAKSCluster --enable-oidc-issuer 
```

## Show the OIDC Issuer URL

To get the OIDC Issuer URL, run the [az aks show][az-aks-show] command. Replace the default values for the cluster name and the resource group name.

```azurecli-interactive
az aks show -n myAKScluster -g myResourceGroup --query "oidcIssuerProfile.issuerUrl" -otsv
```

### Rotate the OIDC key

To rotate the OIDC key, run the [az aks oidc-issuer][az-aks-oidc-issuer] command. Replace the default values for the cluster name and the resource group name.

```azurecli-interactive
az aks oidc-issuer rotate-signing-keys -n myAKSCluster -g myResourceGroup
```

> [!IMPORTANT]
> Once you rotate the key, the old key (key1) expires after 24 hours. This means that both the old key (key1) and the new key (key2) are valid within the 24-hour period. If you want to invalidate the old key (key1) immediately, you need to rotate the OIDC key twice. Then key2 and key3 are valid, and key1 is invalid.

<!-- LINKS - external -->

<!-- LINKS - internal -->
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-update]: /cli/azure/aks#az-aks-update
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-aks-oidc-issuer]: /cli/azure/aks/oidc-issuer