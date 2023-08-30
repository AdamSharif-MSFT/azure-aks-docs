---
title: Long Term Support Azure Kubernetes Service (AKS)
description: Learn about Azure Kubernetes Service (AKS) Long Term Support for Kubernetes
ms.topic: article
ms.date: 08/16/2023

#Customer intent: As a cluster operator or developer, I want to understand how Long Term Support for Kubernetes on AKS works.
---


The Kubernetes community releases a new minor version approximately every three months, with a support window for each version for one year.  This support in terms of AKS is called "Community Support".

AKS will support versions of Kubernetes that are within this Community Support window, as it provides bug fixes, and security updates from the community.

Whilst the innovation that can be delivered with this release cadence provides huge benefits for our customers, it can be challenging to have to keep up to date with Kubernetes releases and this is compounded further by the amount of AKS clusters you have to maintain.  

# Long Term Support

Once a Kubernetes version is out of Community Support, bug fixes and suecurity update will generally not be made available and this leave unsupported Kubernetes clusters at risk.  The AKS team has restarted the upstream LTS working group, and will be maintaing a version of Kubernetes for a period of 2 years.

:::image type="content" source="./media/supported-kubernetes-versions/kubernetes-versions-gantt.png" alt-text="Gantt chart showing the lifecycle of all Kubernetes versions currently active in AKS." lightbox="./media/supported-kubernetes-versions/kubernetes-versions-gantt.png":::

This will be made up of 1 year Community Support and 1 year of Microsoft back porting fixes from the community upstream in our public repository.  Working with the upstream LTS working group, we hope this work can constribute our efforts back to the community whilst providing our customers with the option of a longer support window.

LTS is intended to give customers breathing space to plan their Kubernetes upgrades over a 2 year period.

|   | Community Support  |Long Term Support   |
|---|---|---|
| **When to use** | When you can keep up with upstream Kubernetes releases | When you need control over when to migrate from one version to another  |
|  **Support versions** | Three GA minor versions | One Kubernetes version (currently *1.27*) for two years  |
|  **Pricing** | Included  |  Per hour cluster cost |


## Enabling Long Term Support

We will be maintaining a single version of Kubernetes as LTS at any one time, and this is currently version 1.27.  There is a running cost associated with enabling LTS of $0.60 our and is enabled as part of the Premium tier of AKS.

> [!NOTE]
> Whilst it is possible to enable LTS when the cluster is in Community Support, you will still be charged when you enable to Premium tier.

### Creating a cluster with LTS enabled
```
az aks create --resource-group myResourceGroup --name myAKSCluster --tier premium --k8s-support AKSLongTermSupport
```

> [!NOTE]
> Enabling and disabling LTS is a combination of moving your cluster to the Premium tier, as well as enabling Long Term Support.  Bot must either be turned on or off.

### Enable LTS on an existing cluster
```
az aks update --resource-group myResourceGroup --name myAKSCluster --tier premium --k8s-support AKSLongTermSupport
```

### Disable LTS on an existing cluster
```
az aks update --resource-group myResourceGroup --name myAKSCluster --tier [free|standard] --k8s-support KubernetesCommunitySupport
```

## Long Term Support and Addons
Whilst the AKS team will be maintining the Kubernetes 1.27 code base when it leaves community support, we rely on Open Source projects for the managed add-ons we provide to customers.  Due to various factors, there are some addons that will not be possible to support due to the fact those projects only support Kubernetes versions within upstream community support windows.

As the list of managed add-ons is quite large, we will list the addons that are not supported and the reason why.

|  Add On  | Reason it is unsupported |
---|---|
| Istio |  The Istio support cycle is very short (6 months), and there will not be maintenance releases for k8s 1.27 |
 | Keda | Unable to guarantee future version compatibility with k8s 1.27 |
| Calico  |  Will require Calico Enterprise agreement past community support |
| Cillium  |  Will require Cillium Enterprise agreement past community support |

You will not be able to move a cluster to Long Term support if any of these add-ons are enabled in your cluster.  

Whilst these AKS managed add-ons will not be supported, you will be able to install the Open Source versions of these on your cluster if you wish to use it past Community Support.


## Migrating from LTS 
LTS is meant to extend the support window for customers who need more time to plan their migration to newer Kubernetes releases.  We plan to support a version of Kubernetes for 2 years from General Availability on AKS, and will then mark a later version of Kubernetes as LTS based on:

* Has the previous version had a 2 year support window
* Is there time for customers to migrate from the prior LTS version to the current

The AKS release notes will inform you when we know the two points above are true to plan your migration.

### Migrating from LTS to Communit support
Using LTS as a way to extend your planning to upgrade Kubernetes, it is likely you will want to migrate to a version of Kubernetes that is within the standard support window, something we call N-2 on AKS.  N being the latest Kubernetes 



[add-ons]: integrations.md#add-ons