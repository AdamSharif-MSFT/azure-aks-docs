---
title: Use Azure File with AKS
description: Use Azure Disks with AKS
services: container-service
author: neilpeterson
manager: timlt

ms.service: container-service
ms.topic: article
ms.date: 1/03/2018
ms.author: nepeters
ms.custom: mvc
---

# Persistent volumes with Azure files - dynamic provisioning

A persistent volume represents a piece of storage that has been provisioned for use in a Kubernetes cluster. A persistent volume can be used by one or many pods, and can be dynamically or statically provisioned. For more information on Kubernetes persistent volumes, see [Kubernetes persistent volumes][kubernetes-volumes].

This document details dynamic provisioning of an Azure file share as a Kubernetes persistent volume. 

## Prerequisites

When dynamically provisioning an Azure File share as a Kubernetes volume, any existing storage account can be used as long as it is contained in the same resource group as the AKS cluster. If needed, create a storage account in the AKS resource group. 

To identify the proper resource group, use the [az group list][az-group-list] command.

```azurecli-interactive
az group list --output table
```

The following output shows two resource group associated with an AKS cluster. The resource group with a name like **MC_myAKSCluster_myAKSCluster_eastus** contains the AKS cluster resources, and is where the storage account needs to be created. 

```
Name                                 Location    Status
-----------------------------------  ----------  ---------
MC_myAKSCluster_myAKSCluster_eastus  eastus      Succeeded
myAKSCluster                         eastus      Succeeded
```

Once the resource group has been identified, create the storage account with the [az storage account create][az-storage-account-create] command.

```azurecli-interactive
az storage account create --resource-group  MC_myAKSCluster_myAKSCluster_eastus --name mystorageaccount --location eastus --sku Standard_LRS
```

## Create storage class

A storage class is used to define how a dynamically created persistent volume is configured. Items such as the Azure storage account name, SKU, and region are defined in the storage class object. For more information on Kubernetes storage classes, see [Kubernetes Storage Classes][kubernetes-storage-classes].

The following example specifies that any storage account of sku type `Standard_LRS` in the `eastus` region can be used when requesting storage. 

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: azurefile
provisioner: kubernetes.io/azure-file
parameters:
  skuName: Standard_LRS
```

To use a specific storage account, the `storageAccount` property can be used. In this configuration, the `skuName` and `location` properties are ignored.

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: azurefile
provisioner: kubernetes.io/azure-file
parameters:
  storageAccount: azure_storage_account_name
```

## Create persistent volume claim

A persistent volume claim uses the storage class object to dynamically provision a piece of storage. When using Azure Files, a file share is created in the storage account selected or specified in the storage class object.

The following manifest can be used to create a persistent volume claim `5GB` in size with `ReadWriteOnce` access. For more information on PVC access modes, see [Access Modes][access-modes].

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: azurefile
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: azurefile
  resources:
    requests:
      storage: 5Gi
```

## Using the persistent volume

Once the persistent volume claim has been created, and the storage successfully provisioned, a pod can be created with access to the volume. The following manifest creates a pod that uses the persistent volume claim `azurefile` to mount the Azure File share on the `/var/www/html` path. 

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: volume
  volumes:
    - name: volume
      persistentVolumeClaim:
        claimName: azurefile
```

## Mount options

Default fileMode and dirMode values differ between Kubernetes versions as described in the following table. 

| version | value |
| ---- | ---- |
| v1.6.x, v1.7.x | 0777 |
| v1.8.0-v1.8.5 | 0700 |
| v1.8.6 or above | 0755 |
| v1.9.0 | 0700 |
| v1.9.1 or above | 0755 |

If using a cluster of version 1.8.5 or greater, mount options can be specified on the storage class object. The following example sets `0777`. 

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: azurefile
provisioner: kubernetes.io/azure-file
mountOptions:
  - dir_mode=0777
  - file_mode=0777
  - uid=1000
  - gid=1000
parameters:
  skuName: Standard_LRS
```

If using a cluster of version 1.8.0 - 1.8.4, a security context can be specified with the `runAsUser` value set to `0`. For more information on Pod security context, see [Configure a Security Context][kubernetes-security-context].

## Troubleshooting

If a storage account cannot be found, the persistent volume claim will fail. To get information about the persistent volume claim, used the [kubectl describe][kubectl-describe] command.

First, get the name of the persistent volume claim.

```
kubectl get pvc
```

Output:

```
NAME        STATUS    VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   AGE
azurefile   Pending                                       azurefile      1m
```

Use [kubectl describe][kubectl-describe] to pull information about the current state including any issues.

```
kubectl describe pvc azurefile
```

As can be seen in the output, a suitable storage account was not found. Refer to the prerequisites section of this document for instructions on creating a storage account.

```
Name:          azurefile
Namespace:     default
StorageClass:  azurefile
Status:        Pending
Volume:        
Labels:        <none>
Annotations:   volume.beta.kubernetes.io/storage-provisioner=kubernetes.io/azure-file
Finalizers:    []
Capacity:      
Access Modes:  
Events:
  Type     Reason              Age               From                         Message
  ----     ------              ----              ----                         -------
  Warning  ProvisioningFailed  2s (x11 over 2m)  persistentvolume-controller  Failed to provision volume with StorageClass "azurefile": failed to find a matching storage account
```

## Next steps

Learn more about Kubernetes persistent volumes using Azure Files.

> [!div class="nextstepaction"]
> [Kubernetes plugin for Azure Files][kubernetes-files]

<!-- LINKS - external -->
[access-modes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes
[kubectl-create]: https://kubernetes.io/docs/user-guide/kubectl/v1.8/#create
[kubectl-describe]: https://kubernetes-v1-4.github.io/docs/user-guide/kubectl/kubectl_describe/
[kubernetes-files]: https://github.com/kubernetes/examples/blob/master/staging/volumes/azure_file/README.md
[kubernetes-secret]: https://kubernetes.io/docs/concepts/configuration/secret/
[kubernetes-security-context]: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
[kubernetes-storage-classes]: https://kubernetes.io/docs/concepts/storage/storage-classes/
[kubernetes-volumes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/

<!-- LINKS - internal -->
[az-group-create]: /cli/azure/group#az_group_create
[az-group-list]: /cli/azure/group#az_group_list
[az-storage-account-create]: /cli/azure/storage/account#az_storage_account_create
[az-storage-create]: /cli/azure/storage/account#az_storage_account_create
[az-storage-key-list]: /cli/azure/storage/account/keys#az_storage_account_keys_list
[az-storage-share-create]: /cli/azure/storage/share#az_storage_share_create