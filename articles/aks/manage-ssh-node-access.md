---
title: Manage SSH access on Azure Kubernetes Service cluster nodes 
titleSuffix: Azure Kubernetes Service
description: Learn how to configure SSH on Azure Kubernetes Service (AKS) cluster nodes.
ms.topic: article
ms.date: 08/14/2023
---

# Manage SSH for secure access to Azure Kubernetes Service (AKS) nodes

This article describes how to disable and enable SSH on your AKS clusters or node pools, and how to update the SSH key on your AKS cluster.

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

## Before you begin

* You need the Azure CLI version 2.0.64 or later installed and configured. If you need to install or upgrade, see [Install Azure CLI][install-azure-cli].
* The `aks-preview` Azure CLI extension version 0.5.111 or later. To learn how to install an Azure extension, see [How to install extensions][how-to-install-azure-extensions].
* This feature supports Linux, Mariner, and CBLMariner node pools on new and existing clusters.

## Install the `aks-preview` Azure CLI extension

[!INCLUDE [preview features callout](includes/preview/preview-callout.md)]

1. Install the aks-preview extension using the [`az extension add`][az-extension-add] command.

    ```azurecli
    az extension add --name aks-preview
    ```

2. Update to the latest version of the extension using the [`az extension update`][az-extension-update] command.

    ```azurecli
    az extension update --name aks-preview
    ```

## Register the `DisableSSHPreview` feature flag

1. Register the `DisableSSHPreview` feature flag using the [`az feature register`][az-feature-register] command.

    ```azurecli-interactive
    az feature register --namespace "Microsoft.ContainerService" --name "DisableSSHPreview"
    ```

    It takes a few minutes for the status to show *Registered*.

2. Verify the registration status using the [`az feature show`][az-feature-show] command.

    ```azurecli-interactive
    az feature show --namespace "Microsoft.ContainerService" --name "DisableSSHPreview"
    ```

3. When the status reflects *Registered*, refresh the registration of the *Microsoft.ContainerService* resource provider using the [`az provider register`][az-provider-register] command.

    ```azurecli-interactive
    az provider register --namespace Microsoft.ContainerService
    ```

## Disable SSH overview

To improve security and support your corporate security requirements or strategy, AKS supports disabling SSH (preview) both on the cluster and at the node pool level. Disabling SSH introduces a better approach compared to the only solution, which is to configure [network security group rules][network-security-group-rules-overview] on the AKS subnet/node network interface card (NIC) to restrict specific user outbound IP addresses from connecting to AKS nodes using SSH.

When you disable SSH at cluster creation time, it takes effect after the cluster is created. However, when you disable SSH on an existing node pool, reimage is needed to make the change take effect. After you disable or enable SSH, AKS doesn't automatically reimage your node pool, you can choose any time to perform the reimage operation. Only after reimage is complete, does the disable/enable operation take effect.

A new `securityProfile` variable is added to the `agentPoolProfile` section. It includes a `nodeAccess` property, and `nodeAccess` has the `sshAccess` property, which is an enum type. Current allowed values are `disabled` and `localuser`. `Disabled` means the SSH service is turned off, and `localuser` means the SSH service is on and you can log in as a local user(that is, *azureuser*, *root*, etc.) using a private key (this is the current default behavior).

## Disable SSH on a new cluster deployment

By default, the SSH service on AKS cluster nodes is open to all users and pods running on the cluster. You can prevent direct SSH access from the pod network to the nodes to help limit the attack vector if a container in a pod becomes compromised.

Use the [az aks create][az-aks-create] command to create a new cluster, and include the `--ssh-access disabled` argument to disable SSH (preview) on all the node pools during cluster creation.

```azurecli-interactive
az aks create -g myResourceGroup -n myManagedCluster --ssh-access disabled
```

## Disable SSH on an existing cluster

Use the [az aks update][az-aks-update] command to update an existing cluster, and include the `--ssh-access disabled` argument to disable SSH (preview) on all the node pools in the cluster.

```azurecli-interactive
az aks update -g myResourceGroup -n myManagedCluster --ssh-access disabled
```

For the change to take effect, you need to reimage the node pool by using the [az aks nodepool upgrade][az-aks-nodepool-upgrade] command.

```azurecli-interactive
az aks nodepool upgrade --cluster-name myManagedCluster --name mynodepool --resource-group myResourceGroup --node-image only
```

> [!IMPORTANT]
> During this operation, all Virtual Machine Scale Set instances are upgraded and re-imaged to use the new SSH configuration.

## Disable SSH for a new node pool

Use the [az aks nodepool add][az-aks-nodepool-add] command to add a node pool, and include the `--ssh-access disabled` argument to disable SSH during creation.

```azurecli-interactive
az aks nodepool add --cluster-name myManagedCluster --name mynodepool --resource-group myResourceGroup --ssh-access disabled  
```

The following example output shows that *mynodepool* has been successfully created and SSH is disabled.

```output
```

## Disable SSH for an existing node pool

Use the [az aks nodepool update][az-aks-nodepool-update] command to update a node pool, and include the `--ssh-access disabled` argument to disable SSH (preview).

```azurecli-interactive
az aks nodepool update --cluster-name myManagedCluster --name mynodepool --resource-group myResourceGroup --ssh-access disabled
```

The following example output shows that *mynodepool* has been successfully updated to disable SSH.

```output
```

For the change to take effect, you need to reimage the node pool by using the [az aks nodepool upgrade][az-aks-nodepool-upgrade] command.

```azurecli-interactive
az aks nodepool upgrade --cluster-name myManagedCluster --name mynodepool --resource-group myResourceGroup --node-image only
```

## Update SSH public key on an existing AKS cluster

Use the [az aks update][az-aks-update] command to update the SSH public key (preview) on your cluster. This operation updates the key on all node pools. You can either specify the key or a key file using the `--ssh-key-value` argument.

> [!NOTE]
> Updating of the SSH key is supported on Azure virtual machine scale sets with AKS clusters.

The following are examples of this command:

* To specify the new SSH public key value, include the `--ssh-key-value` argument:

    ```azurecli
    az aks update --name myAKSCluster --resource-group MyResourceGroup --ssh-key-value 'ssh-rsa AAAAB3Nza-xxx'
    ```

* To specify an SSH public key file, specify it with the `--ssh-key-value` argument:

    ```azurecli
    az aks update --name myAKSCluster --resource-group MyResourceGroup --ssh-key-value ~/.ssh/id_rsa.pub
    ```

> [!IMPORTANT]
> During this operation, all Virtual Machine Scale Set instances are upgraded and reimaged to use the new SSH public key.

## Next steps

To help troubleshoot any issues with SSH connectivity to your clusters nodes, you can [view the kubelet logs][view-kubelet-logs] or [view the Kubernetes master node logs][view-master-logs].

<!-- LINKS - external -->

<!-- LINKS - internal -->
[install-azure-cli]: /cli/azure/install-azure-cli
[how-to-install-azure-extensions]: /cli/azure/azure-cli-extensions-overview#how-to-install-extensions
[network-security-group-rules-overview]: concepts-security.md#azure-network-security-groups
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-update]: /cli/azure/aks#az-aks-update
[az-aks-nodepool-update]: /cli/azure/aks/nodepool#az-aks-nodepool-update
[az-feature-register]: /cli/azure/feature#az_feature_register
[az-feature-show]: /cli/azure/feature#az-feature-show
[az-extension-update]: /cli/azure/extension#az_extension_update
[az-provider-register]: /cli/azure/provider#az_provider_register