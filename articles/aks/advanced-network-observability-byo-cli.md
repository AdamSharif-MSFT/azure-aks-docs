---
title: "Setup of Advanced Network Observability for Azure Kubernetes Service (AKS) - BYO Prometheus and Grafana"
description: Get started with Advanced Network Observability for your AKS cluster using BYO Prometheus and Grafana.
author: Khushbu-Parekh
ms.author: kparekh
ms.service: azure-kubernetes-service
ms.subservice: aks-networking
ms.topic: how-to
ms.date: 05/10/2024
ms.custom: template-how-to-pattern, devx-track-azurecli
---

# Setup of Advanced Network Observability for Azure Kubernetes Service (AKS)

Advanced Network Observability is used to collect the network traffic data of your AKS clusters. It enables a centralized platform for monitoring application and network health. Currently, Prometheus collects metrics, and Grafana can be used to visualize them. Advanced Network Observability also offers the ability to enable Hubble. These capabilities are supported for both Cilium and non-Cilium clusters . In this article, learn how to enable these features and use BYO Prometheus and Grafana to visualize the scraped metrics.

For more information about Advanced Network Observability for Azure Kubernetes Service (AKS), see [What is Advanced Network Observability for Azure Kubernetes Service (AKS)?](advanced-container-networking-services-overview.md).

> [!IMPORTANT]
> Advanced Network Observability is currently in PREVIEW.
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## Prerequisites

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

[!INCLUDE [azure-CLI-prepare-your-environment-no-header.md](~/reusable-content/azure-cli/azure-cli-prepare-your-environment-no-header.md)]

- Minimum version of **Azure CLI** required for the steps in this article is **2.56.0**. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI](/cli/azure/install-azure-cli).
### Install the `aks-preview` Azure CLI extension

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
```

### Register the `NetworkObservabilityPreview` feature flag

```azurecli-interactive 
az feature register --namespace "Microsoft.ContainerService" --name "AdvancedNetworkingPreview"
```

Use [az feature show](/cli/azure/feature#az-feature-show) to check the registration status of the feature flag:

```azurecli-interactive
az feature show --namespace "Microsoft.ContainerService" --name "AdvancedNetworkingPreview"
```

Wait for the feature to say **Registered** before preceding with the article.

```output
{
  "id": "/subscriptions/23250d6d-28f0-41dd-9776-61fc80805b6e/providers/Microsoft.Features/providers/Microsoft.ContainerService/features/AdvancedNetworkingPreview",
  "name": "Microsoft.ContainerService/AdvancedNetworkingPreview",
  "properties": {
    "state": "Registering"
  },
  "type": "Microsoft.Features/providers/features"
}
```
When the feature is registered, refresh the registration of the Microsoft.ContainerService resource provider with [az provider register](/cli/azure/provider#az-provider-register):

```azurecli-interactive
az provider register -n Microsoft.ContainerService
```

## Create a resource group

A resource group is a logical container into which Azure resources are deployed and managed. Create a resource group with [az group create](/cli/azure/group#az-group-create) command. The following example creates a resource group named **myResourceGroup** in the **eastus** location:

```azurecli-interactive
az group create \
    --name myResourceGroup \
    --location eastus
```

## Create an AKS cluster with Advanced Network Observability

# [**Non-Cilium**](#tab/non-cilium)

Use [az aks create](/cli/azure/aks#az-aks-create) in the following example to create an AKS cluster with Advanced Network Observability using a non-Cilium dataplane.

```azurecli-interactive
az aks create \
    --name myAKSCluster \
    --resource-group myResourceGroup \
    --location eastus \
    --generate-ssh-keys \
    --network-plugin azure \
    --network-plugin-mode overlay \
    --pod-cidr 192.168.0.0/16 \
    --enable-advanced-network-observability
```

# [**Cilium**](#tab/cilium)

Use [az aks create](/cli/azure/aks#az-aks-create) in the following example to create an AKS cluster with Advanced Network Observability using a Cilium dataplane.

> [!NOTE]
> Clusters with Cilium dataplane support Advanced Observability starting with Kubernetes version 1.29

```azurecli-interactive
az aks create \
    --name myAKSCluster \
    --resource-group myResourceGroup \
    --generate-ssh-keys \
    --location eastus \
    --max-pods 250 \
    --network-plugin azure \
    --network-plugin-mode overlay \
    --network-dataplane cilium \
    --node-count 2 \
    --pod-cidr 192.168.0.0/16 \
    --enable-advanced-network-observability
```

---

## Enable on Existing cluster

Use [az aks update](/cli/azure/aks#az-aks-update) to enable Advanced Network Observability for an existing cluster.

> [!NOTE]
> Clusters with Cilium dataplane support Advanced Observability starting with Kubernetes version 1.29

```azurecli-interactive
az aks update \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --enable-advanced-network-observability
```

## Get cluster credentials 

```azurecli-interactive
az aks get-credentials --name myAKSCluster --resource-group myResourceGroup
```

## Visualization using Grafana

> [!NOTE]
> The following section requires installations of Prometheus and Grafana.

Use the following example to configure scrape jobs on Prometheus and enable visualization on Grafana for your AKS cluster.

1. Add the following scrape job to your existing Prometheus configuration and restart your Prometheus server:

```yml
- job_name: networkobservability-hubble
  kubernetes_sd_configs:
    - role: pod
  relabel_configs:
    - target_label: cluster
      replacement: myAKSCluster
      action: replace
    - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_pod_label_k8s_app]
      regex: kube-system;(retina|cilium)
      action: keep
    - source_labels: [__address__]
      action: replace
      regex: ([^:]+)(?::\d+)?
      replacement: $1:9965
      target_label: __address__
    - source_labels: [__meta_kubernetes_pod_node_name]
      target_label: instance
      action: replace
  metric_relabel_configs:
    - source_labels: [__name__]
      regex: '|hubble_dns_queries_total|hubble_dns_responses_total|hubble_drop_total|hubble_tcp_flags_total' # if desired, add |hubble_flows_processed_total
      action: keep
``` 

1. In **Targets** of Prometheus, verify the **network-obs-pods** are present.

1. Sign in to Grafana and import following example dashboards using the [link](https://grafana.com/grafana/dashboards/?search=Kubernetes+%2F+Networking) or using following ID's:
  * **Clusters:** shows Node-level metrics for your clusters.
  * **DNS (Cluster):** shows DNS metrics on a cluster or selection of Nodes.
  * **DNS (Workload):** shows DNS metrics for the specified workload (e.g. Pods of a DaemonSet or Deployment such as CoreDNS).
  * **Drops (Workload):** shows drops to/from the specified workload (e.g. Pods of a Deployment or DaemonSet).
  * **Pod Flows (Namespace):** shows L4/L7 packet flows to/from the specified namespace (i.e. Pods in the
  Namespace).
  * **Pod Flows (Workload):** shows L4/L7 packet flows to/from the specified workload (e.g. Pods of a Deployment or DaemonSet).

1. In order to access the  data collected by Hubble, install the Hubble CLI:

```azurecli-interactive
HUBBLE_VERSION=0.11
    
HUBBLE_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then HUBBLE_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/hubble/releases/download/$HUBBLE_VERSION/hubble-linux-${HUBBLE_ARCH}.tar.gz{,.sha256sum}
sha256sum --check hubble-linux-${HUBBLE_ARCH}.tar.gz.sha256sum
sudo tar xzvfC hubble-linux-${HUBBLE_ARCH}.tar.gz /usr/local/bin
rm hubble-linux-${HUBBLE_ARCH}.tar.gz{,.sha256sum}
```

## How to Visualize the Hubble Flows

1. Before you move ahead, use the following example to verify the Hubble pods are running. 

```azurecli-interactive
kubectl get po -owide -n kube-system -l k8s-app=hubble-relay
```

```output
hubble-relay-7ddd887cdb-h6khj     1/1  Running     0       23h 
```

1. You will have to port forward Hubble Relay.

```azurecli-interactive
kubectl port-forward -n kube-system svc/hubble-relay --address 127.0.0.1 9999:443
```

1. The Hubble relay server's security is ensured through mutual TLS (mTLS). To enable the Hubble client to retrieve flows, it is necessary to obtain the appropriate certificates and configure the client with these certificates.Use the following commands to apply the certs
```azurecli-interactive
#!/usr/bin/env bash

set -euo pipefail
set -x

# Directory where certificates will be stored
CERT_DIR="$(pwd)/.certs"
mkdir -p "$CERT_DIR"

declare -A CERT_FILES=(
  ["tls.crt"]="tls-client-cert-file"
  ["tls.key"]="tls-client-key-file"
  ["ca.crt"]="tls-ca-cert-files"
)

for FILE in "${!CERT_FILES[@]}"; do
  KEY="${CERT_FILES[$FILE]}"
  JSONPATH="{.data['$FILE']}"

  # Retrieve the secret and decode it
  kubectl get secret hubble-relay-client-certs -n kube-system \
    -o jsonpath="${JSONPATH}" | \
    base64 -d > "$CERT_DIR/$FILE"

  # Set the appropriate hubble CLI config
  hubble config set "$KEY" "$CERT_DIR/$FILE"
done

hubble config set tls true
hubble config set tls-server-name instance.hubble-relay.cilium.io
```

1. Run the following commands to check if the secrets were generated 
```azurecli-interactive
kubectl get po -owide -n kube-system | grep hubble-
```

```output
kube-system     hubble-relay-client-certs     kubernetes.io/tls     3     9d

kube-system     hubble-relay-server-certs     kubernetes.io/tls     3     9d

kube-system     hubble-server-certs           kubernetes.io/tls     3     9d    
 
1. To check that the hubble relay pod is running, run the following command
```azurecli-interactive
hubble relay service 
```

## How to Visualize Using Hubble UI

1. To deploy Hubble UI, save the following into hubble-ui.yaml
```yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: hubble-ui
  namespace: kube-system
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: hubble-ui
  labels:
    app.kubernetes.io/part-of: retina
rules:
  - apiGroups:
      - networking.k8s.io
    resources:
      - networkpolicies
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - componentstatuses
      - endpoints
      - namespaces
      - nodes
      - pods
      - services
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - apiextensions.k8s.io
    resources:
      - customresourcedefinitions
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - cilium.io
    resources:
      - "*"
    verbs:
      - get
      - list
      - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: hubble-ui
  labels:
    app.kubernetes.io/part-of: retina
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: hubble-ui
subjects:
  - kind: ServiceAccount
    name: hubble-ui
    namespace: kube-system
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: hubble-ui-nginx
  namespace: kube-system
data:
  nginx.conf: |
    server {
        listen       8081;
        server_name  localhost;
        root /app;
        index index.html;
        client_max_body_size 1G;
        location / {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            # CORS
            add_header Access-Control-Allow-Methods "GET, POST, PUT, HEAD, DELETE, OPTIONS";
            add_header Access-Control-Allow-Origin *;
            add_header Access-Control-Max-Age 1728000;
            add_header Access-Control-Expose-Headers content-length,grpc-status,grpc-message;
            add_header Access-Control-Allow-Headers range,keep-alive,user-agent,cache-control,content-type,content-transfer-encoding,x-accept-content-transfer-encoding,x-accept-response-streaming,x-user-agent,x-grpc-web,grpc-timeout;
            if ($request_method = OPTIONS) {
                return 204;
            }
            # /CORS
            location /api {
                proxy_http_version 1.1;
                proxy_pass_request_headers on;
                proxy_hide_header Access-Control-Allow-Origin;
                proxy_pass http://127.0.0.1:8090;
            }
            location / {
                try_files $uri $uri/ /index.html /index.html;
            }
            # Liveness probe
            location /healthz {
                access_log off;
                add_header Content-Type text/plain;
                return 200 'ok';
            }
        }
    }
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: hubble-ui
  namespace: kube-system
  labels:
    k8s-app: hubble-ui
    app.kubernetes.io/name: hubble-ui
    app.kubernetes.io/part-of: retina
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s-app: hubble-ui
  template:
    metadata:
      labels:
        k8s-app: hubble-ui
        app.kubernetes.io/name: hubble-ui
        app.kubernetes.io/part-of: retina
    spec:
      serviceAccount: hibble-ui
      serviceAccountName: hubble-ui
      automountServiceAccountToken: true
      containers:
      - name: frontend
        image: mcr.microsoft.com/oss/cilium/hubble-ui:v0.12.2   
        imagePullPolicy: Always
        ports:
        - name: http
          containerPort: 8081
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8081
        readinessProbe:
          httpGet:
            path: /
            port: 8081
        resources: {}
        volumeMounts:
        - name: hubble-ui-nginx-conf
          mountPath: /etc/nginx/conf.d/default.conf
          subPath: nginx.conf
        - name: tmp-dir
          mountPath: /tmp
        terminationMessagePolicy: FallbackToLogsOnError
        securityContext: {}
      - name: backend
        image: mcr.microsoft.com/oss/cilium/hubble-ui-backend:v0.12.2
        imagePullPolicy: Always
        env:
        - name: EVENTS_SERVER_PORT
          value: "8090"
        - name: FLOWS_API_ADDR
          value: "hubble-relay:443"
        - name: TLS_TO_RELAY_ENABLED
          value: "true"
        - name: TLS_RELAY_SERVER_NAME
          value: ui.hubble-relay.cilium.io
        - name: TLS_RELAY_CA_CERT_FILES
          value: /var/lib/hubble-ui/certs/hubble-relay-ca.crt
        - name: TLS_RELAY_CLIENT_CERT_FILE
          value: /var/lib/hubble-ui/certs/client.crt
        - name: TLS_RELAY_CLIENT_KEY_FILE
          value: /var/lib/hubble-ui/certs/client.key
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8090
        readinessProbe:
          httpGet:
            path: /healthz
            port: 8090
        ports:
        - name: grpc
          containerPort: 8090
        resources: {}
        volumeMounts:
        - name: hubble-ui-client-certs
          mountPath: /var/lib/hubble-ui/certs
          readOnly: true
        terminationMessagePolicy: FallbackToLogsOnError
        securityContext: {}
      nodeSelector:
        kubernetes.io/os: linux 
      volumes:
      - configMap:
          defaultMode: 420
          name: hubble-ui-nginx
        name: hubble-ui-nginx-conf
      - emptyDir: {}
        name: tmp-dir
      - name: hubble-ui-client-certs
        projected:
          defaultMode: 0400
          sources:
          - secret:
              name: hubble-relay-client-certs
              items:
                - key: tls.crt
                  path: client.crt
                - key: tls.key
                  path: client.key
                - key: ca.crt
                  path: hubble-relay-ca.crt
---
kind: Service
apiVersion: v1
metadata:
  name: hubble-ui
  namespace: kube-system
  labels:
    k8s-app: hubble-ui
    app.kubernetes.io/name: hubble-ui
    app.kubernetes.io/part-of: retina
spec:
  type: ClusterIP
  selector:
    k8s-app: hubble-ui
  ports:
    - name: http
      port: 80
      targetPort: 8081
```

1. Apply the hubble-ui.yaml manifest to your cluster, using the following command 
```azurecli-interactive
kubectl apply -f hubble-ui.yaml
```
1. Expose Service with port forwarding:
```azurecli-interactive
kubectl port-forward svc/hubble-ui 12000:80
```

1. Now you can access the Hubble UI using your web browser:
http://localhost:12000/

---

## Clean up resources

If you're not going to continue to use this application, delete
the AKS cluster and the other resources created in this article with the following example:

```azurecli-interactive
  az group delete \
    --name myResourceGroup
```

## Next steps

In this how-to article, you learned how to install and enable Advanced Network Observability for your AKS cluster.

- For more information about Advanced Network Observability for Azure Kubernetes Service (AKS), see [What is Advanced Network Observability for Azure Kubernetes Service (AKS)?](advanced-container-networking-services-overview.md).

- To create an AKS cluster with Advanced Network Observability and Azure managed Prometheus and Grafana, see [Setup Advanced Network Observability for Azure Kubernetes Service (AKS) Azure managed Prometheus and Grafana](advanced-network-observability-cli.md).
