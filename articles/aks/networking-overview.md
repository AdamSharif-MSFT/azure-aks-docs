---
title: Network configuration in Azure Kubernetes Service (AKS)
description: Learn about basic and advanced network configuration in Azure Kubernetes Service (AKS).
services: container-service
author: mmacy
manager: jeconnoc

ms.service: container-service
ms.topic: article
ms.date: 05/07/2018
ms.author: marsma
---

# Network configuration in Azure Kubernetes Service (AKS)

When you create an Azure Kubernetes Service (AKS) cluster, you can select from two networking options: **Basic** or **Advanced**.

## Basic networking

The **Basic** networking option is the default configuration for AKS cluster creation. The network configuration of the cluster and its pods are managed completely by Azure, and is appropriate for deployments that do not require custom VNet configuration. You do not have control over network configuration such as subnets or the IP address ranges assigned to the cluster when you select Basic networking.

Nodes in an AKS cluster configured for Basic networking use the [kubenet][kubenet] Kubernetes plugin.

## Advanced networking

**Advanced** networking places your pods in an Azure Virtual Network (VNet) that you configure, providing them automatic connectivity to VNet resources and integration with the rich set of capabilities that VNets offer.

Nodes in an AKS cluster configured for Advanced networking use the [Azure Container Networking Interface (CNI)][cni-networking] Kubernetes plugin.

![Diagram showing two nodes with bridges connecting each to a single Azure VNet][advanced-networking-diagram-01]

The Azure CNI plugin is also supported by the open source [Azure Container Service Engine (ACS Engine)][acs-engine] project.

## Advanced networking features

Advanced networking provides the following benefits:

* You can deploy your AKS cluster into an existing VNet, or create a new VNet and subnet for your cluster.
* Every pod in the cluster is assigned an IP address in the VNet, and can directly communicate with other pods in the cluster, and other VMs in the VNet.
* A pod can connect to other services in a peered VNet, and to on-premises networks over ExpressRoute and site-to-site (S2S) VPN connections. Pods are also reachable from on-premises.
* A Kubernetes service can be exposed externally or internally through the Azure Load Balancer.
* Pods in a subnet that have service endpoints enabled can securely connect to Azure services, for example Azure Storage and SQL DB.
* You can use user-defined routes (UDR) to route traffic from pods to a Network Virtual Appliance.
* Pods can access resources on the public Internet.

## Configure advanced networking

When you [create an AKS cluster](kubernetes-walkthrough-portal.md) in the Azure portal, the following parameters are configurable for advanced networking:

**Virtual network**: The VNet into which you want to deploy the Kubernetes cluster. If you want to create a new VNet for your cluster, select *Create new* and follow the steps in the *Create virtual network* section.

**Subnet**: The subnet within the VNet where you want to deploy the cluster. If you want to create a new subnet in the VNet for your cluster, select *Create new* and follow the steps in the *Create subnet* section.

**Kubernetes service address range**: The IP address range for the Kubernetes cluster's service IPs. This range must not be within the VNet IP address range of your cluster.

**Kubernetes DNS service IP address**:  The IP address for the cluster's DNS service. This address must be within the *Kubernetes service address range*.

**Docker Bridge address**: The IP address and netmask to assign to the Docker bridge. This IP address must not be within the VNet IP address range of your cluster.

The following screenshot from the Azure portal shows an example of configuring these settings during AKS cluster creation:

![Advanced networking configuration in the Azure portal][portal-01-networking-advanced]

## Plan IP addressing for your cluster

Clusters configured with Advanced networking require additional planning. The size of your VNet and its subnet must accommodate the number of pods you plan to run simultaneously in the cluster, as well as your scaling requirements.

Each VNet provisioned for use with the Azure CNI plugin is limited to 4096 IP addresses.

IP addresses for the pods and the cluster's nodes are assigned from the specified subnet within the VNet. Each node is configured with a primary IP, which is the IP of the node itself, and 30 additional IP addresses pre-configured by Azure CNI that are assigned to pods scheduled to the node. When you scale out your cluster, each node is similarly configured with IP addresses from the subnet.

Each node in a cluster configured for Advanced networking can host a maximum of 30 pods.

## Frequently asked questions

The following questions and answers apply to the **Advanced** networking configuration.

* *Can I deploy VMs in my cluster subnet?*

  Yes, you can. However, ensure you have a sufficient number of IP addresses in the subnet for the VMs.

* *Are there any scenarios in which Network Security Groups, user-defined routes, and other network policies will not work for pods?*

  Per-pod network policies are currently unsupported. You can configure the policies, but their behavior may be unpredictable, and they may not be functional. As such, their usage is discouraged.

* *Is the maximum number of pods deployable to a node configurable?*

  Each node can host a maximum of 30 pods. This number is not currently configurable.

* *How do I configure additional properties for the subnet that I created during AKS cluster creation? For example, service endpoints.*

  The complete list of properties for the VNet and subnets that you create during AKS cluster creation can be configured in the standard VNet configuration page in the Azure portal.

## Next steps

Learn more about networking in AKS in the following articles:

[Use a static IP address with the Azure Kubernetes Service (AKS) load balancer](static-ip.md)

[HTTPS ingress on Azure Container Service (AKS)](ingress.md)

[Use an internal load balancer with Azure Container Service (AKS)](internal-lb.md)

<!-- IMAGES -->
[advanced-networking-diagram-01]: ./media/networking-overview/advanced-networking-diagram-01.png
[portal-01-networking-advanced]: ./media/networking-overview/portal-01-networking-advanced.png

<!-- LINKS - External -->
[acs-engine]: https://github.com/Azure/acs-engine
[cni-networking]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[kubenet]: https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#kubenet

<!-- LINKS - Internal -->
[aks-ssh]: aks-ssh.md
