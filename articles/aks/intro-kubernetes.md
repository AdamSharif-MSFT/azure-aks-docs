---
title: Introduction to Azure Kubernetes Service
description: Azure Kubernetes Service makes it simple to deploy and manage container-based applications on Azure.
services: container-service
author: neilpeterson
manager: jeconnoc

ms.service: container-service
ms.topic: overview
ms.date: 06/13/2018
ms.author: nepeters
ms.custom: mvc
---

# Azure Kubernetes Service (AKS)

Azure Kubernetes Service (AKS) makes it simple to create, configure, and manage a cluster of virtual machines preconfigured to run containerized applications. AKS reduces the complexity and operational overhead of managing a Kubernetes cluster by offloading much of that responsibility to Azure. As a hosted Kubernetes service, Azure handles critical tasks like health monitoring and maintenance for you. In addition the service is free, you only pay for the agent nodes within your clusters, not for the masters.

This document provides an overview on the features of Azure Kubernetes Service (AKS).

## Certification and compliance

Azure Kubernetes Service has been CNCF certified as Kubernetes conformant. AKS is also complaint with SOC and ISO/HIPPA/HITRUST.

![CErtified Kubernetes](media/acs-intro/certified-kubernetes.svg)

## Flexible deployment options

Azure Kubernetes Service offers portal, command line, and template driven deployment options (ARM templates and Terraform). When deploying an AKS cluster, the Kubernetes master and all nodes are deployed and configured for you. Additional features such as advanced networking, Azure Active Directory integration, and monitoring can also be configured during the deployment process.

For more information, see both the [AKS portal quickstart][aks-portal] or the [AKS CLI quickstart][aks-cli].

## Identity and security management

AKS clusters are RBAC enabled by default. An AKS cluster can also be configured to integrate with Azure Active Directory. In this configuration, Kubernetes access can be configured based on Azure Active Directory identity and group membership.

For more information, see, [Integrate Azure Active Directory with AKS][aks-aad].

## Integrated logging and monitoring

Container health gives you performance visibility by collecting memory and processor metrics from containers, nodes, and controllers. Container logs are also collected. This data is stored in your Log Analytics workspace, and is available through the Azure portal, Azure CLI, or a REST endpoint.

For more information, see [Monitor Azure Kubernetes Service container health][container-health].

## Cluster node scaling

As demand for resources increases, the nodes of an AKS cluster can be scaled out to match. If resource demand drops, nodes can be removed by scaling in the cluster. AKS scale operations can be completed using the Azure portal or the Azure CLI.

For more information, see [Scale an Azure Kubernetes Service (AKS) cluster][aks-scale].

## Cluster node upgrades

Azure Kubernetes Service offers multiple Kubernetes versions. As new versions become available in AKS, your cluster can be upgraded using the Azure portal or Azure CLI. During the upgrade process, nodes are carefully cordoned and drained to minimize disruption to running application.

For more information, see [Upgrade an Azure Kubernetes Service (AKS) cluster][aks-upgrade].

## HTTP application routing

The HTTP Application Routing solution makes it easy to access applications deployed to your AKS cluster. When enabled, the HTTP application routing solution configures an ingress controller in your AKS cluster. As applications are deployed, publically accessible DNS names are auto configured.

For more information, see [HTTP application routing][aks-http-routing].

## GPU enabled nodes

AKS supports the creation of GPU enabled node pools. Azure currently provides single or multiple GPU enabled VMs. GPU enabled VMs are designed for compute-intensive, graphics-intensive, and visualization workloads.

For more information, see [Using GPUs on AKS][aks-gpu].

## Development tooling integration

Kubernetes has a rich ecosystem of development and management tools such as Helm, Draft, and the Kubernetes extension for Visual Studio Code. These tools work seamlessly with Azure Kuberntees Service.

Additionally, Azure Dev Spaces provides a rapid, iterative Kubernetes development experience for teams. With minimal configuration, you can run and debug containers directly in Azure Kubernetes Service (AKS).

For more information, see [Azure Dev Spaces][azure-dev-spaces]

Azure DevOps project provides a simple solution for bringing existing code and Git repository into Azure. The DevOps project automatically creates Azure resources such as AKS, a release pipeline in VSTS that includes a build definition for CI, sets up a release definition for CD, and then creates an Azure Application Insights resource for monitoring.

For more information, see [Azure DevOps project][azure-devops]

## Private container registry

Integrate with Azure Container Registry (ACR) for private storage of your Docker images.

For more information, see [Azure Container Registry (ACR)][acr-docs].

## Next steps

Learn more about deploying and managing AKS with the AKS quickstart.

> [!div class="nextstepaction"]
> [AKS Tutorial][aks-cli]

<!-- LINKS - external -->
[acs-engine]: https://github.com/Azure/acs-engine
[draft]: https://github.com/Azure/draft
[helm]: https://helm.sh/
[kubectl-overview]: https://kubernetes.io/docs/user-guide/kubectl-overview/

<!-- LINKS - internal -->
[azure-dev-spaces]: https://docs.microsoft.com/en-us/azure/dev-spaces/azure-dev-spaces
[azure-devops]: https://docs.microsoft.com/en-us/vsts/pipelines/actions/azure-devops-project-aks?view=vsts
[acr-docs]: ../container-registry/container-registry-intro.md
[aks-aad]: ./aad-integration.md
[aks-cli]: ./kubernetes-walkthrough.md
[aks-gpu]: ./gpu-cluster.md
[aks-http-routing]: ./http-application-routing.md
[aks-portal]: ./kubernetes-walkthrough-portal.md
[aks-scale]: ./scale-cluster.md
[aks-upgrade]: ./upgrade-cluster.md
[container-health]: ../monitoring/monitoring-container-health.md

