---
title: Use a customer-managed key to encrypt Azure disks in Azure Kubernetes Service (AKS)
description: Bring your own keys (BYOK) to encrypt AKS OS and Data disks.
services: container-service
author: mlearned

ms.service: container-service
ms.topic: article
ms.date: 01/01/2020
ms.author: mlearned
---

# Bring your own keys (BYOK) with Azure disks in Azure Kubernetes Service (AKS)

Azure Storage encrypts all data in a storage account at rest. By default, data is encrypted with Microsoft-managed keys. For additional control over encryption keys, you can supply [customer-managed keys][https://docs.microsoft.com/azure/virtual-machines/windows/disk-encryption#customer-managed-keys-public-preview] to use for encryption of both the OS and data disks for your AKS clusters.

> [!NOTE]
> Linux and Windows based AKS clusters are both supported.

## Before you begin

* This article assumes that you are creating a new AKS cluster.  You will also need to use or create an instance of Azure Key Vault to store your encryption keys.

* You must enable soft delete and purge protection for Azure Key Vault when using Key Vault to encrypt managed disks.

* You need the Azure CLI version 2.0.77 or later and the aks-preview 0.4.26 extension

> [!IMPORTANT]
> AKS preview features are self-service opt-in. Previews are provided "as-is" and "as available" and are excluded from the service level agreements and limited warranty. AKS Previews are partially covered by customer support on best effort basis. As such, these features are not meant for production use. For additional infromation, please see the following support articles:
>
> * [AKS Support Policies](support-policies.md)
> * [Azure Support FAQ](faq.md)


## Current supported regions

* Australia East
* Canada Central, Canada East
* Central US, East US, East US 2, North Central US, South Central US
* West Central US, West US, West US 2
* East Asia, South East Asia
* France, North Europe, UK South, West Europe

## Install latest AKS CLI preview extension

To use customer-managed keys, you need the *aks-preview* CLI extension version 0.4.26 or higher. Install the *aks-preview* Azure CLI extension using the [az extension add][az-extension-add] command, then check for any available updates using the [az extension update][az-extension-update] command::

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview

## Create Azure Key Vault instance to store your keys

You can optionally use the Azure portal to [Configure customer-managed keys with Azure Key Vault][https://docs.microsoft.com/azure/storage/common/storage-encryption-keys-portal]

Create a new Key Vault instance and enable soft delete and purge protection.

```azurecli-interactive
az keyvault create -n <key-vault-name> -g <resource-group-name> -l <azure-location-name>  --enable-purge-protection true --enable-soft-delete true
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/AKSPrivateLinkPreview')].{Name:name,State:properties.state}"
```

## Create an instance of a DiskEncryptionSet
    
```azurecli
keyVaultId=$(az keyvault show --name <key-vault-name> --query [id] -o tsv)
keyVaultKeyUrl=$(az keyvault key show --vault-name <key-vault-name>  --name <key-name>  --query [key.kid] -o tsv)
az disk-encryption-set create -n <disk-encryption-set-name>  -l <azure-location-name>  -g <resource-group-name> --source-vault <key-vault-id> --key-url <key-vault-url> 
```

## Grant the DiskEncryptionSet resource access to the key vault

Use the DiskEncryptionSet and resource groups you created on the prior steps, and grant the DiskEncryptionSet resource access to the Azure Key Vault.

```azurecli
desIdentity=$(az disk-encryption-set show -n <disk-encryption-set-name>  -g <resource-group-name> --query [identity.principalId] -o tsv)
az keyvault set-policy -n <key-vault-name> -g <resource-group-name> --object-id $desIdentity --key-permissions wrapkey unwrapkey get
az role assignment create --assignee $desIdentity --role Reader --scope $keyVaultId
```

## Create an AKS cluster with and encrypt the OS disk with a customer-manged key

Create a new AKS cluster and use your key to encrypt the OS disk.

```azurecli-interactive
diskEncryptionSetId=$(az resource show -n $diskEncryptionSetName -g ssecmktesting --resource-type "Microsoft.Compute/diskEncryptionSets" --query [id] -o tsv)
az group create -n resourceGroupName -l westcentralus
az aks create -n clusterName -g resourceGroupName --node-osdisk-diskencryptionsetid diskEncryptionId
```

## Encrypt an AKS cluster data disk with a customer-managed key

## Limitations

* OS Disk Encryption supported with Kubernetes version 1.17 and above   
* Available only in regions where BYOK is supported
* This is currently for new AKS clusters only, existing clusters cannot be upgraded
* AKS cluster using Virtual Machine Scale Sets are required, no support Virtual Machine Availablity Sets


## Next steps

TODO

<!-- LINKS - external -->
[access-modes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubernetes-storage-classes]: https://kubernetes.io/docs/concepts/storage/storage-classes/
[kubernetes-volumes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/
[managed-disk-pricing-performance]: https://azure.microsoft.com/pricing/details/managed-disks/

<!-- LINKS - internal -->
[azure-disk-volume]: azure-disk-volume.md
[azure-files-pvc]: azure-files-dynamic-pv.md
[premium-storage]: ../virtual-machines/windows/disks-types.md
[az-disk-list]: /cli/azure/disk#az-disk-list
[az-snapshot-create]: /cli/azure/snapshot#az-snapshot-create
[az-disk-create]: /cli/azure/disk#az-disk-create
[az-disk-show]: /cli/azure/disk#az-disk-show
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[operator-best-practices-storage]: operator-best-practices-storage.md
[concepts-storage]: concepts-storage.md
[storage-class-concepts]: concepts-storage.md#storage-classes
