---
title: Use KMS etcd encryption in Azure Kubernetes Service (AKS) 
description: Learn how to use kms etcd encryption with Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 07/04/2022

---

# Add KMS etcd encryption to an Azure Kubernetes Service (AKS) cluster 

This article shows you how to enable encryption at rest for your Kubernetes data in etcd using Azure Key Vault with Key Management Service (KMS) plugin. The KMS plugin allows you to:

* Use a key in Key Vault for etcd encryption
* Bring your own keys
* Provide encryption at rest for secrets stored in etcd
* Rotate the keys in Key Vault

For more information on using the KMS plugin, see [Encrypting Secret Data at Rest](https://kubernetes.io/docs/tasks/administer-cluster/kms-provider/).

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

## Before you begin

* An Azure subscription. If you don't have an Azure subscription, you can create a [free account](https://azure.microsoft.com/free).
* [Azure CLI installed](/cli/azure/install-azure-cli).


## Limitations

The following limitations apply when you integrate KMS etcd encryption with AKS:

* Deletion of the key, Key Vault, or the associated identity.
* KMS etcd encryption doesn't work with System-Assigned Managed Identity. The keyvault access-policy is required to be set before the feature is enabled. In addition, System-Assigned Managed Identity isn't available until cluster creation, thus there's a cycle dependency. 
* Using more than 2000 secrets in a cluster.
* Bring your own (BYO) Azure Key Vault from another tenant.

KMS supports [public key vault](Enable KMS with public key vault) and [public key vault](Enable KMS with private key vault) now. 

## Enable KMS with public key vault

### Create a key vault and key

> [!WARNING]
> Deleting the key or the Azure Key Vault is not supported and will cause your cluster to become unstable.
> 
> If you need to recover your Key Vault or key, see the [Azure Key Vault recovery management with soft delete and purge protection](../key-vault/general/key-vault-recovery.md?tabs=azure-cli) documentation.

Use `az keyvault create` to create a KeyVault.

```azurecli
az keyvault create --name MyKeyVault --resource-group MyResourceGroup
```

Use `az keyvault key create` to create a key.

```azurecli
az keyvault key create --name MyKeyName --vault-name MyKeyVault
```

Use `az keyvault key show` to export the Key ID.

```azurecli
export KEY_ID=$(az keyvault key show --name MyKeyName --vault-name MyKeyVault --query 'key.kid' -o tsv)
echo $KEY_ID
```

The above example stores the Key ID in *KEY_ID*.
 
### Create a user-assigned managed identity

Use `az identity create` to create a User-assigned managed identity.

```azurecli
az identity create --name MyIdentity --resource-group MyResourceGroup
```

Use `az identity show` to get Identity Object ID.

```azurecli
IDENTITY_OBJECT_ID=$(az identity show --name MyIdentity --resource-group MyResourceGroup --query 'principalId' -o tsv)
echo $IDENTITY_OBJECT_ID
```

The above example stores the value of the Identity Object ID in *IDENTITY_OBJECT_ID*.

Use `az identity show` to get Identity Resource ID.

```azurecli
IDENTITY_RESOURCE_ID=$(az identity show --name MyIdentity --resource-group MyResourceGroup --query 'id' -o tsv)
echo $IDENTITY_RESOURCE_ID
```

The above example stores the value of the Identity Resource ID in *IDENTITY_RESOURCE_ID*.

### Assign permissions (decrypt and encrypt) to access key vault

Use `az keyvault set-policy` to create an Azure KeyVault policy.

```azurecli-interactive
az keyvault set-policy -n MyKeyVault --key-permissions decrypt encrypt --object-id $IDENTITY_OBJECT_ID
```

### Create an AKS cluster with KMS etcd encryption enabled

Create an AKS cluster using the [az aks create][az-aks-create] command with the `--enable-azure-keyvault-kms` and `--azure-keyvault-kms-key-id` parameters to enable KMS etcd encryption.

```azurecli-interactive
az aks create --name myAKSCluster --resource-group MyResourceGroup --assign-identity $IDENTITY_RESOURCE_ID --enable-azure-keyvault-kms --azure-keyvault-kms-key-id $KEY_ID
```

### Update an exiting AKS cluster to enable KMS etcd encryption

Use [az aks update][az-aks-update] with the `--enable-azure-keyvault-kms` and `--azure-keyvault-kms-key-id` parameters to enable KMS etcd encryption on an existing cluster.

```azurecli-interactive
az aks update --name myAKSCluster --resource-group MyResourceGroup --enable-azure-keyvault-kms --azure-keyvault-kms-key-id $KEY_ID
```

Use below command to update all secrets. Otherwise, the old secrets aren't encrypted. 

```azurecli-interactive
kubectl get secrets --all-namespaces -o json | kubectl replace -f -
```

### Rotate the existing keys 
After changing the key ID (including key name and key version), you could use [az aks update][az-aks-update] with the `--enable-azure-keyvault-kms` and `--azure-keyvault-kms-key-id` parameters to rotate the exitsing keys of KMS.

```azurecli-interactive
az aks update --name myAKSCluster --resource-group MyResourceGroup  --enable-azure-keyvault-kms --azure-keyvault-kms-key-id $NewKEY_ID 
```

Use below command to update all secrets. Otherwise, the old secrets are still encrypted with the previous key. 

```azurecli-interactive
kubectl get secrets --all-namespaces -o json | kubectl replace -f -
```

## Enable KMS with private key vault

AKS cluster will create a private endpoint and private link in the node resource group automatically. The key vault will be added a private endpoint connection with the AKS cluster.

### Create a private key vault and key

> [!WARNING]
> Deleting the key or the Azure Key Vault is not supported and will cause your cluster to become unstable.
> 
> If you need to recover your Key Vault or key, see the [Azure Key Vault recovery management with soft delete and purge protection](../key-vault/general/key-vault-recovery.md?tabs=azure-cli) documentation.

Without private endpoint, we could not create or update keys in private key vault. Thus we would create a key vault and key first, then update the key vault to private mode. For existing private key vault, you could refer to [Integrate Key Vault with Azure Private Link](private-link-service.md).

Use `az keyvault create` to create a public KeyVault.

```azurecli
az keyvault create --name MyKeyVault --resource-group MyResourceGroup 
```

Use `az keyvault show` to export the resource ID of key vault.

```azurecli
export KEYVAULT_RESOURCE_ID=$(az keyvault show --name MyKeyVault --query 'id' -o tsv)
echo $KEYVAULT_RESOURCE_ID
```

The above example stores the key vault resource ID in *KEYVAULT_RESOURCE_ID*.

Use `az keyvault key create` to create a key.

```azurecli
az keyvault key create --name MyKeyName --vault-name MyKeyVault
```

Use `az keyvault key show` to export the Key ID.

```azurecli
export KEY_ID=$(az keyvault key show --name MyKeyName --vault-name MyKeyVault --query 'key.kid' -o tsv)
echo $KEY_ID
```

The above example stores the Key ID in *KEY_ID*.

Use `az keyvault update` to disable public access of existing key vault:

```azurecli
az keyvault update --name MyKeyVault --resource-group MyResourceGroup --public-network-access Disable
```

### Create a user-assigned managed identity

Use `az identity create` to create a User-assigned managed identity.

```azurecli
az identity create --name MyIdentity --resource-group MyResourceGroup
```

Use `az identity show` to get Identity Object ID.

```azurecli
IDENTITY_OBJECT_ID=$(az identity show --name MyIdentity --resource-group MyResourceGroup --query 'principalId' -o tsv)
echo $IDENTITY_OBJECT_ID
```

The above example stores the value of the Identity Object ID in *IDENTITY_OBJECT_ID*.

Use `az identity show` to get Identity Resource ID.

```azurecli
IDENTITY_RESOURCE_ID=$(az identity show --name MyIdentity --resource-group MyResourceGroup --query 'id' -o tsv)
echo $IDENTITY_RESOURCE_ID
```

The above example stores the value of the Identity Resource ID in *IDENTITY_RESOURCE_ID*.

### Assign permissions (decrypt and encrypt) to access key vault

Use `az keyvault set-policy` to create an Azure KeyVault policy.

```azurecli-interactive
az keyvault set-policy -n MyKeyVault --key-permissions decrypt encrypt --object-id $IDENTITY_OBJECT_ID
```

For private key vault, the AKS needs *Key Vault Contributor* to create private link between private key vault and cluster.

```azurecli-interactive
az role assignment create --role “Key Vault Contributor” --assignee-object-id $IDENTITY_OBJECT_ID --assignee-principal-type "ServicePrincipal" --scope $KEYVAULT_RESOURCE_ID
```

### Create an AKS cluster with private key vault and enable KMS etcd encryption 

Create an AKS cluster using the [az aks create][az-aks-create] command with the `--enable-azure-keyvault-kms`, `--azure-keyvault-kms-key-id`, `--azure-keyvault-kms-key-vault-network-access` and `--azure-keyvault-kms-key-vault-resource-id` parameters to enable KMS etcd encryption with private key vault.

```azurecli-interactive
az aks create --name myAKSCluster --resource-group MyResourceGroup --assign-identity $IDENTITY_RESOURCE_ID --enable-azure-keyvault-kms --azure-keyvault-kms-key-id $KEY_ID --azure-keyvault-kms-key-vault-network-access "Private" --azure-keyvault-kms-key-vault-resource-id $KEYVAULT_RESOURCE_ID
```

### Update an exiting AKS cluster to enable KMS etcd encryption with private key vault

Use [az aks update][az-aks-update] with the `--enable-azure-keyvault-kms`, `--azure-keyvault-kms-key-id`, `--azure-keyvault-kms-key-vault-network-access` and `--azure-keyvault-kms-key-vault-resource-id` parameters to enable KMS etcd encryption on an existing cluster with private key vault.

```azurecli-interactive
az aks update --name myAKSCluster --resource-group MyResourceGroup --enable-azure-keyvault-kms --azure-keyvault-kms-key-id $KEY_ID --azure-keyvault-kms-key-vault-network-access "Private" --azure-keyvault-kms-key-vault-resource-id $KEYVAULT_RESOURCE_ID
```

Use below command to update all secrets. Otherwise, the old secrets aren't encrypted. 

```azurecli-interactive
kubectl get secrets --all-namespaces -o json | kubectl replace -f -
```

### Rotate the existing keys 
After changing the key ID (including key name and key version), you could use [az aks update][az-aks-update] with the `--enable-azure-keyvault-kms` and `--azure-keyvault-kms-key-id` parameters to rotate the exitsing keys of KMS.

```azurecli-interactive
az aks update --name myAKSCluster --resource-group MyResourceGroup  --enable-azure-keyvault-kms --azure-keyvault-kms-key-id $NewKEY_ID --azure-keyvault-kms-key-vault-network-access "Private" --azure-keyvault-kms-key-vault-resource-id $KEYVAULT_RESOURCE_ID
```

Use below command to update all secrets. Otherwise, the old secrets are still encrypted with the previous key. 

```azurecli-interactive
kubectl get secrets --all-namespaces -o json | kubectl replace -f -
```

## Disable KMS

Use below command to disable KMS on existing cluster.

```azurecli-interactive
az aks update --name myAKSCluster --resource-group MyResourceGroup --disable-azure-keyvault-kms
```



<!-- LINKS - Internal -->
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az extension add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-extension-add]: /cli/azure/extension#az_extension_add
[az-extension-update]: /cli/azure/extension#az_extension_update
[az-feature-register]: /cli/azure/feature#az_feature_register
[az-feature-list]: /cli/azure/feature#az_feature_list
[az-provider-register]: /cli/azure/provider#az_provider_register
[az-aks-update]: /cli/azure/aks#az_aks_update
[Enable KMS with public key vault]:use-kms-etcd-encryption.md#enable-kms-with-public-key-vault
[Enable KMS with private key vault]:use-kms-etcd-encryption.md#enable-kms-with-private-key-vault
