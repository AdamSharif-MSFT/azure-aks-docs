---
title: Use pod security policies in Azure Kubernetes Service (AKS)
description: Learn how to control pod admissions by using PodSecurityPolicy in Azure Kubernetes Service (AKS)
services: container-service
author: iainfoulds

ms.service: container-service
ms.topic: article
ms.date: 02/01/2019
ms.author: iainfou
---

# Limit   using pod security policies in Azure Kubernetes Service (AKS)

This article shows you how to use pod security policies to limit the deployment of pods in AKS.

> [!IMPORTANT]
> This feature is currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use][terms-of-use]. Some aspects of this feature may change prior to general availability (GA).

## Before you begin

This article assumes that you have an existing AKS cluster. If you need an AKS cluster, see the AKS quickstart [using the Azure CLI][aks-quickstart-cli] or [using the Azure portal][aks-quickstart-portal].

You need the Azure CLI version 2.0.56 or later installed and configured. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI][install-azure-cli].

To create or update an AKS cluster to use pod security policies, first enable a feature flag on your subscription. To register the *EnablePodSecurityPolicy* feature flag, use the [az feature register][az-feature-register] command as shown in the following example:

```azurecli-interactive
az feature register --name EnablePodSecurityPolicy --namespace Microsoft.ContainerService
```

It takes a few minutes for the status to show *Registered*. You can check on the registration status using the [az feature list][az-feature-list] command:

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/EnablePodSecurityPolicy')].{Name:name,State:properties.state}"
```

When ready, refresh the registration of the *Microsoft.ContainerService* resource provider using the [az provider register][az-provider-register] command:

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

## Overview of pod security policies

In a Kubernetes cluster, an admission controller is used to intercept requests to the API server when a resource is to be created. The admission controller can then *validate* the resource request against a set of rules, or *mutate* the resource to change deployment parameters.

*PodSecurityPolicy* is an admission controller that validates a pod specification meets your defined requirements. These requirements may limit the use of privileged containers, access to certain types of storage, or the user or group the container can run as. When you try to deploy a resource where the pod specifications don't meet the requirements outlined in the pod security policy, the request is denied. This ability to control what pods can be scheduled in the AKS cluster prevents some possible security vulnerabilities or privilege escalations.

## Test the creation of a privileged pod

Before you enable PodSecurityPolicy, let's first test what happens when a pod with the security context of `privileged: true` is scheduled. In the next step, a policy is created to block this type of request in a pod specification. Create a file named `nginx-privileged.yaml` and paste the following YAML manifest.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-privileged
spec:
  containers:
    - name: nginx-privileged
      image: nginx:1.14.2
      securityContext:
        privileged: true
```

Create the pod using the [kubectl apply][kubectl-apply] command and specify the name of your YAML manifest:

```console
kubectl apply -f nginx-privileged.yaml
```

The pod is successfully scheduled, as shown in the following example output:

```console
$ kubectl apply -f nginx-privileged.yaml

pod/nginx-privileged created

$ kubectl get pods

NAME               READY   STATUS    RESTARTS   AGE
nginx-privileged   1/1     Running   0          4m51s
```

Before you move on, delete this pod using [kubectl delete][kubectl-delete] command and specify the name of your YAML manifest:

```console
kubectl delete -f nginx-privileged.yaml
```

## Create a pod security policy

Before you can enable pod security policy in an AKS cluster, you first define what these policies are. Without policies in place, no pods can be created in your cluster. Let's create a policy that prevents a pod that requests privileged access that was scheduled in the previous step. Create a file named `psp-deny-privileged.yaml` and paste the following YAML manifest:

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: psp-deny-privileged
spec:
  privileged: false
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  runAsUser:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  volumes:
  - '*'
```

Create the policy using the [kubectl apply][kubectl-apply] command and specify the name of your YAML manifest:

```console
kubectl apply -f psp-deny-privileged.yaml
```

***-- ANYTHING ELSE NEEDED HERE ? --***

## Allow user account to use the pod security policy

In the previous step, you created a pod security policy that won't schedule pods that request privileged access. Associate this policy with a user account using a *RoleBinding* or *ClusterRoleBinding*. These bindings are typically created at a group level, and give you the ability to target pod security policies at a specific namespace or across the whole cluster.

***-- ADD STEPS ON HOW TO CREATE ROLEBINDING. OR, DO A CLUSTERROLEBINDING? USE A YAML MANIFEST FOR THIS, OR JUST CREATE THE BINDING DIRECTLY? --***

## Enable pod security policy on the AKS cluster

With policies created in your AKS cluster, you can enable pod security policy using the [az aks update-cluster][az-aks-update-cluster] command. The following example enables pod security policy on the cluster name *myAKSCluster* in the resource group named *myResourceGroup*:

```azurecli-interactive
az aks update-cluster \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --enable-pod-security-policy
```

## Test the creation of privileged pod again

With a pod security policy in place, and a binding for the service account to use the policy, let's try to create a privileged pod again. Use the same *nginx-privileged.yaml* manifest to create the pod using the [kubectl apply][kubectl-apply] command:

```console
kubectl apply -f nginx-privileged.yaml
```

The pod fails to be scheduled when the pod security policy admission controller validates the requests in the pod specification. As the pod specification requested privileged escalation, the *psp-deny-privileged* policy denies the request before the API server creates the resource.

***-- SHOW EXAMPLE OF WHAT THE FAILURE LOOKS LIKE --***

## Clean up resources

Delete the failed NGINX privileged pod created in the previous step using [kubectl delete][kubectl-delete] command and specify the name of your YAML manifest:

```console
kubectl delete -f nginx-privileged.yaml
```

To disable pod security policy, use the [az aks update-cluster][az-aks-update-cluster] command. The following example enables pod security policy on the cluster name *myAKSCluster* in the resource group named *myResourceGroup*:

```azurecli-interactive
az aks update-cluster \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --disable-pod-security-policy
```

***-- DELETE THE ROLEBINDING / CLUSTERROLEBINDING --***

Finally, delete the network policy using [kubectl delete][kubectl-delete] command and specify the name of your YAML manifest:

```console
kubectl delete -f psp-deny-privileged.yaml
```

## Next steps

For more information about network resources, see [Network concepts for applications in Azure Kubernetes Service (AKS)][concepts-network].

<!-- LINKS - external -->
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-delete]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - internal -->
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[concepts-network]: concepts-network.md
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
