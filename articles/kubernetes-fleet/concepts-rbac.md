---
title: "Grant access to Azure Kubernetes Fleet Manager resources with Azure role-based access control"
description: This article provides an overview of the Azure role-based access control roles that can be used to access Azure Kubernetes Fleet Manager resources.
ms.date: 04/29/2024
author: nickomang
ms.author: nickoman
ms.service: kubernetes-fleet
ms.topic: conceptual
---

# Grant access to Azure Kubernetes Fleet Manager resources with Azure role-based access control

[Azure role-based access control (Azure RBAC)][azure-rbac-overview] is an authorization system built on Azure Resource Manager that provides fine-grained access management to Azure resources.

This article provides an overview of the various built-in Azure RBAC roles that you can use to access Azure Kubernetes Fleet Manager (Fleet) resources.

## Control plane

This role grants access to Azure Resource Manager (ARM) Fleet resources and sub-resources, and is applicable both to hubless and hubful Fleets.


|Role name|Description|Usage|
|---------|-----------|-----|
|[Azure Kubernetes Fleet Manager Contributor][azure-rbac-fleet-manager-contributor-role]|This role grants read and write access to Azure resources provided by Azure Kubernetes Fleet Manager, including fleets, fleet members, fleet update strategies, fleet update runs, and more.|You can use this role to grant Contributor permissions that apply solely to Fleet resources and sub-resources. For example, this role can be given to an Azure administrator tasked with defining and maintaining Fleet resources.|

- **Azure Kubernetes Fleet Manager Contributor Role**

    This role grants read and write access to Azure resources provided by Azure Kubernetes Fleet Manager, including fleets, fleet members, fleet update strategies, fleet update runs, and more.

    You can use this role to grant Contributor permissions that apply solely to Fleet resources and sub-resources. For example, this role can be given to an Azure administrator tasked with defining and maintaining Fleet resources.

    For more details, see [Azure Kubernetes Fleet Manager Contributor Role][azure-rbac-fleet-manager-contributor-role].

## Data plane

These roles grant access to Fleet hub Kubernetes objects, and are therefore only applicable to hubful Fleets.

You can assign data plane roles at the Fleet hub cluster scope, or at an individual Kubernetes namespace scope by appending `/namespace/<namespace>` to the role assignment scope.

|Role name|Description|Usage|
|---------|-----------|-----|
|[Azure Kubernetes Fleet Manager RBAC Reader][azure-rbac-fleet-manager-rbac-reader]|Grants read-only access to most Kubernetes resources within a namespace in the fleet-managed hub cluster. It does not allow viewing roles or role bindings. This role does not allow viewing Secrets, since reading the contents of Secrets enables access to `ServiceAccount` credentials in the namespace, which would allow API access as any `ServiceAccount` in the namespace (a form of privilege escalation). Applying this role at cluster scope will give access across all namespaces.|You can use this role to grant the capability to read selected non-sensitive Kubernetes objects at either namespace or cluster scope. For example, you can grant this role for review purposes.|
|[Azure Kubernetes Fleet Manager RBAC Writer][azure-rbac-fleet-manager-rbac-writer]|Grants read and write access to most Kubernetes resources within a namespace in the fleet-managed hub cluster. This role does not allow viewing or modifying roles or role bindings. However, this role allows accessing Secrets as any `ServiceAccount` in the namespace, so it can be used to gain the API access levels of any `ServiceAccount` in the namespace. Applying this role at cluster scope will give access across all namespaces.|You can use this role to grant the capability to write selected Kubernetes objects at either namespace or cluster scope. For example, for use by a project team responsible for objects in a given namespace.|
|[Azure Kubernetes Fleet Manager RBAC Admin][azure-rbac-fleet-manager-rbac-admin]|Grants read and write access to Kubernetes resources within a namespace in the fleet-managed hub cluster. Provides write permissions on most objects within a namespace, with the exception of `ResourceQuota` object and the namespace object itself. Applying this role at cluster scope will give access across all namespaces.|You can use this role to grant the capability to administer selected Kubernetes objects (including roles and role bindings) at either namespace or cluster scope. For example, for use by a project team responsible for objects in a given namespace.|
|[Azure Kubernetes Fleet Manager RBAC Cluster Admin][azure-rbac-fleet-manager-rbac-cluster-admin]|Grants read/write access to all Kubernetes resources in the fleet-managed hub cluster.|You can use this role to grant access to all Kubernetes objects (including CRDs) at either namespace or cluster scope.|

## Example role assignments

You can grant Azure RBAC roles using the [Azure CLI][azure-cli-overview]. For example, to create a role assignment at the Fleet hub cluster scope:

```azurecli-interactive
IDENTITY=$(az ad signed-in-user show --output tsv --query id)
FLEET_ID=$(az fleet show --resource-group $GROUP --name $FLEET --output tsv --query id)

az role assignment create --role 'Azure Kubernetes Fleet Manager RBAC Reader' --assignee "$IDENTITY" --scope "$FLEET_ID"
```

You can also scope role assignments to an individual Kubernetes namespace. For example, to create a role assignment for a Fleet hub's default Kubernetes namespace:

```azurecli-interactive
IDENTITY=$(az ad signed-in-user show --output tsv --query id)
FLEET_ID=$(az fleet show --resource-group $GROUP --name $FLEET --output tsv --query id)

az role assignment create --role 'Azure Kubernetes Fleet Manager RBAC Reader' --assignee "$IDENTITY" --scope "$FLEET_ID/namespaces/default"
```

<!-- LINKS -->
[azure-cli-overview]: /cli/azure/what-is-azure-cli.md
[azure-rbac-overview]: /azure/role-based-access-control/overview.md
[azure-rbac-fleet-manager-contributor-role]: /azure/role-based-access-control/built-in-roles/containers.md#azure-kubernetes-fleet-manager-contributor-role
[azure-rbac-fleet-manager-rbac-reader]: /azure/role-based-access-control/built-in-roles/containers.md#azure-kubernetes-fleet-manager-rbac-reader
[azure-rbac-fleet-manager-rbac-writer]: /azure/role-based-access-control/built-in-roles/containers.md#azure-kubernetes-fleet-manager-rbac-writer
[azure-rbac-fleet-manager-rbac-admin]: /azure/role-based-access-control/built-in-roles/containers.md#azure-kubernetes-fleet-manager-rbac-admin
[azure-rbac-fleet-manager-rbac-cluster-admin]: /azure/role-based-access-control/built-in-roles/containers.md#azure-kubernetes-fleet-manager-rbac-cluster-admin