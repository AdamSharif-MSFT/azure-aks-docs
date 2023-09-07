---
title: Use Kubernetes events for troubleshooting
description: Learn about Kubernetes events, which provide details on pods, nodes, and other Kubernetes objects.
ms.topic: article
ms.author: nickoman
author: nickomang
ms.date: 09/07/2023
---

# Kubernetes events for troubleshooting

Events are one of the prominent sources for monitoring and troubleshooting issues in Kubernetes. They capture and record information about the lifecycle of various Kubernetes objects, such as pods, nodes, services, and deployments. By monitoring events, you can gain visibility into your cluster's activities, identify issues, and troubleshoot problems effectively.

Kubernetes events do not persist throughout your cluster life cycle, as there is no mechanism for retention. They are short-lived, only available for one hour after the event is generated. To store events for a longer time period, enable [Container Insights][container-insights].

## Kubernetes event objects

Below is a set of the important fields in a Kubernetes Event. For a comprehensive list of all fields, check the official [Kubernetes documentation][k8s-events].

|Field name|Significance|
|----------|------------|
|type |Significance changes based on the severity of the event:<br/>**Warning:** these events signal potentially problematic situations, such as a pod repeatedly failing or a node running out of resources. They require attention, but might not result in immediate failure.<br/>**Normal:** These events represent routine operations, such as a pod being scheduled or a deployment scaling up. They usually indicate healthy cluster behavior.|
|reason|The reason why the event was generated. For example, *FailedScheduling* or *CrashLoopBackoff*.|
|message|A human-readable message that describes the event.|
|namespace|The namespace of the Kubernetes object that the event is associated with.|
|firstSeen||
|lastSeen||
|reportingController|The name of the controller that reported the event. For example, `kubernetes.io/kubelet`|
|object|The name of the Kubernetes object that the event is associated with.|

## Accessing events

<!-- # [Azure CLI](#tab/azure-cli/linux) -->

You can find events for your cluster and its components by using `kubectl`. Keep in mind the Kubernetes API server only retains events for 1 hour after the last occurrence.

```azurecli-interactive
kubectl get events
```

To look at a specific pod's events, first find the name of the pod and then use `kubectl describe` to list events.

```azurecli-interactive
kubectl get pods

kubectl describe pod <pod-name>
```

<!--

# [Portal](#tab/azure-portal)

 Content for portal -->

--

## Best practices for troubleshooting with events

### Filtering events for relevance

In your AKS cluster, you might have various namespaces and services running. Filtering events based on object type, namespace, or reason can help you narrow down your focus to what's most relevant to your applications. For instance, you can use the following command to filter events within a specific namespace:

```azurecli-interactive
kubectl get events -n <namespace>
```

### Automating event notifications

To ensure timely response to critical events in your AKS cluster, set up automated notifications. Azure offers integration with monitoring and alerting services like [Azure Monitor][aks-azure-monitor]. You can configure alerts to trigger based on specific event patterns. This way, you're immediately informed about crucial issues that require attention.

### Regularly reviewing events

Make a habit of regularly reviewing events in your AKS cluster. This proactive approach can help you identify trends, catch potential problems early, and prevent escalations. By staying on top of events, you can maintain the stability and performance of your applications.

## Next steps

Now that you understand Kubernetes events, you can continue your monitoring and observability journey by [enabling Container Insights][container-insights].

<!-- LINKS -->
[aks-azure-monitor]: ./monitor-aks.md
[container-insights]: ../azure-monitor/containers/container-insights-enable-aks.md
[k8s-events]: https://kubernetes.io/docs/reference/kubernetes-api/cluster-resources/event-v1/
