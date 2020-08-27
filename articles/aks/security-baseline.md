---
title: Azure security baseline for Azure Kubernetes Service
description: The Azure Kubernetes Service security baseline provides procedural guidance and resources for implementing the security recommendations specified in the Azure Security Benchmark.
author: msmbaldwin
ms.service: container-service
ms.topic: conceptual
ms.date: 08/26/2020
ms.author: mbaldwin
ms.custom: security-benchmark

# Important: This content is machine generated; do not modify this topic directly. Contact mbaldwin for more information.

---

# Azure security baseline for Azure Kubernetes Service

The Azure Security Baseline for Azure Kubernetes Service contains recommendations that will help you improve the security posture of your deployment.

The baseline for this service is drawn from the [Azure Security Benchmark version 1.0](../security/benchmarks/overview.md), which provides recommendations on how you can secure your cloud solutions on Azure with our best practices guidance.

For more information, see [Azure Security Baselines overview](../security/benchmarks/security-baselines-overview.md).

>[!WARNING]
>This preview version of the article is for review only. **DO NOT MERGE INTO MASTER!**

## Network security

*For more information, see the [Azure Security Benchmark: Network security](/azure/security/benchmarks/security-control-network-security).*

### 1.1: Protect Azure resources within virtual networks

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32877.).

**Guidance**: Use network security groups to filter the flow of traffic in virtual networks with defined source and destination IP ranges, allowed ports and protocols or deny access to resources such as Microsoft Azure Kubernetes Service (AKS) nodes. 

Review automatic protections built in to AKS. By default, a network security group and route table are automatically created with the creation of an Microsoft Azure Kubernetes Service (AKS) cluster. The network security group is then automatically associated with the virtual network interface cards on the customer nodes and the route table is associated with the subnet on the virtual network. As more services are created and exposed, the network security group rules and route tables are also automatically updated. 

AKS automatically modifies the network security groups for appropriate traffic flow as services are created with load balancers, port mappings, or ingress routes. Default rules are created to allow Transport Layer security (TLS) traffic to the Kubernetes API server. As services, such as a Load Balancer are created, the Azure platform automatically configures the appropriate rules as part of a Kubernetes Service manifest where you could define required ports and forwarding. 

Deploy AKS clusters into an existing Azure virtual network subnet if one already exists. 

Use a private AKS cluster to ensure network traffic between your AKS API server and your node pools remains only on the private network, as the control plane or API server always uses internal (RFC1918) IP addresses. 

No public IP addresses are assigned for AKS nodes deployed into a private virtual network subnet.  
Choose to allow or deny specific network paths within the cluster based on choice of namespaces and label selectors using network policies. 

The control plane or API server of an AKS cluster is in an AKS-managed Azure subscription while a customer's cluster or node pool is in the customer's subscription. The server and the cluster or node pool communicate with each other through the Azure Private Link service in the API server virtual network and a private endpoint that's exposed in the customer's AKS cluster subnet.

By default, AKS clusters have unrestricted outbound (egress) internet access which allows running nodes and services to access external resources as needed. 
To restrict egress traffic, a limited number of ports and addresses must be accessible to maintain healthy cluster maintenance tasks due to outbound dependencies on services outside of that virtual network. These outbound dependencies are almost entirely defined with FQDNs, which don't have static addresses behind them. That means NSGs cannot be used to lock down the outbound traffic from an AKS cluster. 

Do not manually configure NSG rules to filter traffic for pods in an AKS cluster. The cluster does not have inbound dependencies. 

Use Kubernetes network policies in AKS to limit network traffic between pods in an AKS cluster. AKS clusters use kubenet with an Azure virtual network and subnet created for the customers. AKS kubenet nodes get an IP address from the Azure virtual network subnet with network policies. Every pod gets an IP address from the subnet and can be accessed directly with Azure Container Networking Interface (CNI). 

Bring your own existing subnet and route table as these are supported by AKS. A route table must exist on your cluster subnets with kubenet. AKS creates a route table if one does not exist in a custom subnet and adds rules to it throughout the cluster lifecycle. If a custom subnet does contain a route table, when a cluster is created, AKS acknowledges the existing route table during cluster operations and adds or updates rules accordingly for cloud provider operations.

- [Create an AKS cluster in the virtual network](configure-kubenet.md#create-an-aks-cluster-in-the-virtual-network)

- [Bring your own subnet and route table with kubenet](configure-kubenet.md#bring-your-own-subnet-and-route-table-with-kubenet)

- [How to configure networking for your AKS instance](configure-azure-cni.md#configure-networking---portal)

- [Understand network security configurations for AKS](concepts-network.md#azure-virtual-networks)

- [Configure Azure CNI networking in Azure Kubernetes Service (AKS)](configure-azure-cni.md)

- [Network concepts for applications in Azure Kubernetes Service (AKS)](concepts-network.md)

- [Use Kubenet networking with your own IP address ranges in Azure Kubernetes Service (AKS)](configure-kubenet.md)

- [Security concepts for applications and clusters in Azure Kubernetes Service (AKS)](concepts-security.md)

Control egress traffic for cluster nodes in Azure Kubernetes Service (AKS) limit-egress-traffic.md

- [Secure traffic between pods using network policies in Azure Kubernetes Service (AKS)](use-network-policies.md)

- [Create a private Azure Kubernetes Service cluster](private-clusters.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 1.2: Monitor and log the configuration and traffic of virtual networks, subnets, and NICs

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32878.).

**Guidance**: Use Security Center and follow its' network protection recommendations to help secure the network resources being used by your Azure Kubernetes Service (AKS) clusters. 

Enable network security group flow logs for your AKS management virtual network/subnet and send NSG flow logs to an Azure Log Analytics workspace. 

Use Traffic Analytics to provide insights into traffic flow in your Azure cloud. Azure Traffic Analytics provides visual network activity, hot spots and security threats identification, traffic flow patterns, and pinpoints network misconfigurations.

- [How to Enable NSG Flow Logs](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [How to Enable and use Traffic Analytics](../network-watcher/traffic-analytics.md)

- [Understand Network Security provided by Azure Security Center](../security-center/security-center-network-recommendations.md)

- [Understand best practices for network connectivity and security in AKS](operator-best-practices-network.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 1.3: Protect critical web applications

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32879.).

**Guidance**: Use a Web Application Firewall (WAF) which provides an additional layer of security by filtering the incoming traffic to your web applications. Azure WAF uses a set of rules to watch for attacks like cross site scripting or cookie poisoning against this traffic. These rules are provided by The Open Web Application Security Project (OWASP) and updated regularly.

Consider implementing Kubernetes ingress resources and controllers, operating at layer 7  (Application layer), of the Open systems Interconnect (OSI) model, for web applications that use HTTP or HTTPS protocols.
 
Create an Application Gateway Ingress Controller (AGIC) for Azure Kubernetes Service (AKS) customers, to implement Azure's native Application Gateway L7 load-balancer to expose cloud software to the Internet.

Use an API gateway for authentication, authorization, throttling, caching, transformation, and monitoring for APIs used in your AKS environment. An API gateway serves as a front door to the microservices, decouples clients from your microservices, adds an additional layer of security, and decreases the complexity of your microservices by removing the burden of handling cross cutting concerns.

- [Understand best practices for network connectivity and security in AKS](operator-best-practices-network.md)

- [Application Gateway Ingress Controller ](../application-gateway/ingress-controller-overview.md)

- [How to get started by creating an Application Gateway Ingress Controller](https://github.com/azure/application-gateway-kubernetes-ingress)

- [Use Azure API Management with microservices deployed in Azure Kubernetes Service](../api-management/api-management-kubernetes.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 1.4: Deny communications with known malicious IP addresses

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32880.).

**Guidance**: Enable DDoS Standard protection on the Virtual Networks where Azure Kubernetes Service (AKS) components are deployed for protections against distributed denial-of-service (DDoS) attacks
Use network policies to allow or deny traffic to pods. By default, all traffic is allowed between pods within a cluster. Define rules that limit pod communication for improved security. Choose to allow or deny traffic based on settings such as assigned labels, namespace, or traffic port. The required network policies can be automatically applied as pods are dynamically created in an AKS cluster. Do not use Azure NSGs to control pod-to-pod traffic, use network policies instead. 

- [Secure traffic between pods using network policies in Azure Kubernetes Service (AKS)](use-network-policies.md)

- [How to configure DDoS protection](../virtual-network/manage-ddos-protection.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 1.5: Record network packets

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32881.).

**Guidance**: Enable network security group flow logs and send the logs to an Azure Storage account for auditing. You can also send the flow logs to a Log Analytics workspace and then use Traffic Analytics to provide insights into traffic patterns in your Azure cloud. Some advantages of Traffic Analytics are the ability to visualize network activity, identify hot spots and security threats, understand traffic flow patterns, and pinpoint network misconfigurations. 

Enable Network Watcher packet capture, if required for investigating anomalous activity. 

Additionally, the netstat utility can introspect a wide variety of network stats, invoked with summary output. There are many useful fields depending on the issue. One useful field in the TCP section is "failed connection attempts". This may be an indication of SNAT port exhaustion or other issues making outbound connections. A high rate of retransmitted segments (also under the TCP section) may indicate issues with packet delivery.

- [How to Enable NSG Flow Logs](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [How to enable Network Watcher](../network-watcher/network-watcher-create.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 1.6: Deploy network-based intrusion detection/intrusion prevention systems (IDS/IPS)

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32882.).

**Guidance**: Secure your Azure Kubernetes Service (AKS) cluster with a Azure Application Gateway with web application firewall (WAF). 

If intrusion detection and/or prevention based on payload inspection or behavior analytics is not a requirement, Azure Application Gateway with WAF can be used and configured in "detection mode" to log alerts and threats, or "prevention mode" to actively block detected intrusions and attacks.

The Application Gateway Ingress Controller (AGIC) add-on allows AKS customers to leverage Azure's native Application Gateway level 7 load-balancer to expose cloud software to the Internet. AGIC monitors the Kubernetes cluster it is hosted on and continuously updates an Application Gateway, so that selected services are exposed to the Internet. 

The Ingress Controller runs in its own pod on the customer’s AKS. AGIC monitors a subset of Kubernetes Resources for changes. The state of the AKS cluster is translated to Application Gateway specific configuration and applied to the Azure Resource Manager (ARM).

- [Application Gateway ingress controller overview](../application-gateway/ingress-controller-overview.md)

- [Understand best practices for securing your AKS cluster with a WAF](operator-best-practices-network.md#secure-traffic-with-a-web-application-firewall-waf)

- [How to deploy Azure Application Gateway (Azure WAF)](../web-application-firewall/ag/application-gateway-web-application-firewall-portal.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 1.7: Manage traffic to web applications

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32883.).

**Guidance**: Use network security groups to filter the flow of traffic in virtual networks with defined source and destination IP ranges, allowed ports and protocols or deny access to resources such as Microsoft Azure Kubernetes Service (AKS)  nodes. 

Implement network policies to allow or deny traffic to pods. By default, all traffic is allowed between pods within a cluster. Define rules that limit pod communication for improved security. Choose to allow or deny traffic based on settings such as assigned labels, namespace, or traffic port. The required network policies can be automatically applied as pods are dynamically created in an AKS cluster. Do not use Azure NSGs to control pod-to-pod traffic, use network policies instead. 

Use a web application firewall (WAF) for an additional layer of security. A WAF can filter the incoming traffic and can be placed in front of an Azure Kubernetes Service (AKS) Cluster. The Open Web Application Security Project (OWASP) provides a set of rules which are used in the WAF to watch for attacks like cross site scripting or cookie poisoning.
Consider implementing Kubernetes ingress resources and controllers, operating at layer 7  (Application layer of the Open systems Interconnect (OSI) model), for web applications that use HTTP or HTTPS protocols. This allows Transmission Layer Security (TLS) termination to be handled by the Ingress resource rather than within the application itself on large web applications accessed via HTTPS, offloading traffic saturation concerns. Configure the Ingress resource to use providers such as Let's Encrypt for automatic TLS certification generation and configuration.

Create an Application Gateway Ingress Controller (AGIC) for Azure Kubernetes Service (AKS) customers, to implement Azure's native Application Gateway L7 load-balancer to expose cloud software to the Internet.

Apply Fully Qualified Domain Name (FQDN) tags to applications for ease of use in setting up application rules within an network security group. 

After setting up the network rules. add an application rule using an FQDN tag, for example, AzureKubernetesService, which includes all required FQDNs accessible through TCP port 443 and port 80. 

- [Understand best practices for network connectivity and security in AKS](operator-best-practices-network.md)

- [Application Gateway ingress controller overview](../application-gateway/ingress-controller-overview.md)

- [How to get started by creating an Application Gateway Ingress Controller](https://github.com/azure/application-gateway-kubernetes-ingress)

- [Secure traffic between pods using network policies in Azure Kubernetes Service (AKS)](use-network-policies.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 1.8: Minimize complexity and administrative overhead of network security rules

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32884.).

**Guidance**: Use virtual network service tags to define network access controls on network security groups associated with your Azure Kubernetes Service (AKS) instances. 

Use service tags in place of specific IP addresses when creating security rules within an Network Security Group. 

Allow or deny the traffic for the corresponding service by specifying the service tag name (for example, ApiManagement) in the appropriate source or destination field of a rule. 

Microsoft manages the address prefixes encompassed by the service tag and automatically updates the service tag as addresses change.

Apply an Azure tag to node pools in your AKS cluster. Note that the tags applied to a node pool are different than the virtual network service tags, and are applied to each node within the node pool and persist through upgrades. 

Adding a tag can help with tasks such as policy tracking or cost estimation as tags are also applied to new nodes added to a node pool during scale-out operations. 

Apply Fully Qualified Domain Name (FQDN) tags to applications for ease of use in setting up application rules within an network security group. 

After setting up the network rules. add an application rule using an FQDN tag, for example, AzureKubernetesService, which includes all required FQDNs accessible through TCP port 443 and port 80. 

- [Understand and using Service Tags](../virtual-network/service-tags-overview.md)

- [Understand NSGs for AKS](support-policies.md)

- [Control egress traffic for cluster nodes in Azure Kubernetes Service (AKS)](limit-egress-traffic.md)

- [Secure traffic between pods using network policies in Azure Kubernetes Service (AKS)](use-network-policies.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 1.9: Maintain standard security configurations for network devices

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32885.).

**Guidance**: Define and implement standard security configurations with Azure Policy for network resources associated with your Azure Kubernetes Service (AKS) clusters. 
Use Azure Policy aliases in the "Microsoft.ContainerService" and "Microsoft.Network" namespaces to create custom policies to audit or enforce the network configuration of your AKS clusters. 

Also, use of built-in policy definitions related to AKS, such as:

•	Authorized IP ranges should be defined on Kubernetes Services

•	Enforce HTTPS ingress in Kubernetes cluster

•	Ensure services listen only on allowed ports in Kubernetes cluster

- [How to configure and manage Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Azure Policy samples for networking](/azure/governance/policy/samples/#network)

- [Understand Azure Policy for Kubernetes clusters (preview)](../governance/policy/concepts/policy-for-kubernetes.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 1.10: Document traffic configuration rules

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32886.).

**Guidance**: Use tags for network security groups and other resources for network security and traffic flow to and from Azure Kubernetes Service (AKS) clusters. Use the "Description" field for individual network security group rules to specify business need and/or duration, and so on, for any rules that allow traffic to/from a network.
Use any of the built-in Azure policy tagging-related definitions, for example, "Require tag and its value" that ensure that all resources are created with tags and to receive notifications for existing untagged resources.

Implement the Network Policy feature in Kubernetes to define rules for ingress and egress traffic between pods in an AKS cluster. 

Choose to allow or deny specific network paths within the cluster based on namespaces and label selectors with network policies. 
Use these namespaces and labels as descriptors for traffic configuration rules.

Apply Fully Qualified Domain Name (FQDN) tags to applications for ease of use in setting up application rules within an network security group. 

After setting up the network rules. add an application rule using an FQDN tag, for example, AzureKubernetesService, which includes all required FQDNs accessible through TCP port 443 and port 80 

You may use Azure PowerShell or Azure command-line interface (CLI) to look-up or perform actions on resources based on their Tags.

- [Azure Policy with CLI](https://docs.microsoft.com/cli/azure/policy?view=azure-cli-latest)

- [How to create and use tags](/azure/azure-resource-manager/resource-group-using-tags)

- [How to create a Virtual Network](../virtual-network/quick-create-portal.md)

- [How to create an NSG with a Security Config](../virtual-network/tutorial-filter-network-traffic.md)

- [Secure traffic between pods using network policies in Azure Kubernetes Service (AKS)](use-network-policies.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 1.11: Use automated tools to monitor network resource configurations and detect changes

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32887.).

**Guidance**: Use Azure Activity Log to monitor network resource configurations and detect changes for network resources related to Azure Kubernetes Service (AKS) clusters. 

Create alerts within Azure Monitor that will trigger when changes to critical network resources take place. Note that the any entries from the AzureContainerService user in the activity logs are logged as platform actions. 

Use Azure Monitor logs to enable and query the logs from AKS the master components, kube-apiserver and kube-controller-manager. 
Create and manage the nodes that run the kubelet with container runtime and deploy their applications through the managed Kubernetes API server. 

- [How to view and retrieve Azure Activity Log events](/azure/azure-monitor/platform/activity-log-view)

- [How to create alerts in Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

- [Enable and review Kubernetes master node logs in Azure Kubernetes Service (AKS)](view-master-logs.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

## Logging and monitoring

*For more information, see the [Azure Security Benchmark: Logging and monitoring](/azure/security/benchmarks/security-control-logging-monitoring).*

### 2.1: Use approved time synchronization sources

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32888.).

**Guidance**: Azure Kubernetes Service (AKS) nodes use ntp.ubuntu.com for time synchronization, along with UDP port 123 and Network Time Protocol (NTP). 

Ensure NTP servers are accessible by the cluster nodes if using custom DNS servers. 

- [Understand NTP domain and port requirements for AKS cluster nodes](limit-egress-traffic.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Shared

### 2.2: Configure central security log management

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32889.).

**Guidance**: Azure Kubernetes Services'(AKS) master components, kube-apiserver and kube-controller-manager are provided as a managed service. You have to create and manage the nodes that run the kubelet and container runtime, and deploy your applications through the managed Kubernetes API server. 

Enable audit logs, such as: 

•	kube-auditaksService: The display name in audit log for the control plane operation (from the hcpService) 

•	masterclient: The display name in audit log for MasterClientCertificate, the certificate you get from az aks get-credentials 

•	nodeclient: The display name for ClientCertificate, which is used by agent nodes

Enable other audit logs such as kube-audit as well. 

- [Note that the Log schema including log roles are available for review here](view-master-logs.md)

- [Understand Azure Monitor for Containers](../azure-monitor/insights/container-insights-overview.md)

- [How to enable Azure Monitor for Containers](../azure-monitor/insights/container-insights-onboard.md)

- [Enable and review Kubernetes master node logs in Azure Kubernetes Service (AKS)](view-master-logs.md)

**Azure Security Center monitoring**: Yes

**Responsibility**: Customer

### 2.3: Enable audit logging for Azure resources

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32890.).

**Guidance**: Azure Monitor logs work with both RBAC and non-RBAC enabled AKS clusters.

Configure Diagnostic Settings to enable log collection for the Kubernetes master components in your Azure Kubernetes Service (AKS) cluster.

Enable audit logs, such as: 

•	kube-auditaksService: The display name in audit log for the control plane operation (from the hcpService) 

•	masterclient: The display name in audit log for MasterClientCertificate, the certificate you get from az aks get-credentials 

•	nodeclient: The display name for ClientCertificate, which is used by agent nodes

Enable other audit logs such as kube-audit as well. 

- [How to enable and review Kubernetes master node logs in AKS](view-master-logs.md)

**Azure Security Center monitoring**: Yes

**Responsibility**: Customer

### 2.4: Collect security logs from operating systems

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32891.).

**Guidance**: Enable automatic provisioning of the Azure Log Analytics Agent. 
Security center collects data from Azure virtual machines (VMs), virtual machine scale sets, and IaaS containers, such as Kubernetes cluster nodes, to monitor for security vulnerabilities and threats. Data is collected using the Azure Log Analytics Agent, which reads various security-related configurations and event logs from the machine and copies the data to your workspace for analysis. 

Data collection is required to provide visibility into missing updates, misconfigured OS security settings, endpoint protection status, and health and threat detections.

- [How to enable automatic provisioning of the Log Analytics Agent](../security-center/security-center-enable-data-collection.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 2.5: Configure security log storage retention

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32892.).

**Guidance**: After onboarding your Azure Kubernetes Service (AKS) instances to Azure Monitor for containers, set the corresponding Azure Log Analytics workspace retention period according to your organization's compliance regulations.

- [How to set log retention parameters for Log Analytics Workspaces](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 2.6: Monitor and review Logs

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32893.).

**Guidance**: Onboard your Azure Kubernetes Service (AKS) instances to Azure Monitor for containers and configure diagnostic settings for your cluster. 

Use Azure Monitor's Log Analytics workspace to review logs and perform queries on log data. Azure Monitor logs are enabled and managed in the Azure portal, or through CLI, and work with both RBAC and non-RBAC enabled AKS clusters.

View the logs generated by these master components (kube-apiserver and kube-controllermanager) for troubleshooting your application and services.

Enable and on-board data to Azure Sentinel or a third party SIEM for centralized log management and monitoring.

- [How to enable and review Kubernetes master node logs in AKS](view-master-logs.md)

- [How to onboard Azure Sentinel](../sentinel/quickstart-onboard.md)

- [How to perform custom queries in Azure Monitor](../azure-monitor/log-query/get-started-queries.md)

**Azure Security Center monitoring**: Yes

**Responsibility**: Customer

### 2.7: Enable alerts for anomalous activities

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32894.).

**Guidance**: Use Azure Kubernetes Service (AKS) together with Security Center to gain deeper visibility into AKS nodes. 
Security Center alerts you to threats and malicious activity detected at the host and at the cluster level. It implements continuous analysis of raw security events occurring in an AKS cluster, such as network data, process creation, and the Kubernetes audit log. 

Determine if this activity is expected behavior or whether the application is misbehaving. Use metrics and logs in Azure Monitor to substantiate your findings. An example metric is: "Failed" category for Source Network Address Translation (SNAT) Connections, which can point to network resource exhaustion issues.

- [Understand Azure Kubernetes Services integration with Security Center](../security-center/azure-kubernetes-service-integration.md)

- [How to enable Azure Security Center Standard Tier](../security-center/security-center-get-started.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 2.8: Centralize anti-malware logging

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32895.).

**Guidance**: Not applicable to Azure Kubernetes Service (AKS). Search the Azure marketplace for a containerized 3rd party anti-malware solution which supports clusters.

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 2.9: Enable DNS query logging

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32896.).

**Guidance**: Azure Kubernetes Service (AKS) uses the CoreDNS project for cluster DNS management and resolution with all 1.12.x and higher clusters. 

DNS query logging can be enabled by applying documented configuration in your coredns-custom ConfigMap. Refer to noted article for details.

- [Customize CoreDNS with Azure Kubernetes Service](coredns-custom.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 2.10: Enable command-line audit logging

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32897.).

**Guidance**: Not applicable to Azure Kubernetes Service (AKS) service. AKS nodes are managed and not generally accessible by customers.

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

## Identity and access control

*For more information, see the [Azure Security Benchmark: Identity and access control](/azure/security/benchmarks/security-control-identity-access-control).*

### 3.1: Maintain an inventory of administrative accounts

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32898.).

**Guidance**: Azure Kubernetes Service (AKS) itself does not provide an identity management solution that stores regular user accounts and passwords. Instead, external identity solutions, such as Azure Active Directory (Azure AD) should be integrated into AKS. 
Grant users or groups access to Kubernetes resources within a namespace or across the cluster using Azure AD integration. 

Invoke the Azure AD PowerShell module to perform ad hoc queries to discover accounts that are members of AKS administrative groups  

Azure CLI for operations can be used to ‘Get access credentials for a managed Kubernetes cluster’ which can assist in reconciling access on a regular basis.

Implement Security Center Identity and Access Management recommendations.

Configure a service principal manually or use an existing service principal to authenticate ACR (Azure Container Resources) from AKS, instead of using an Owner or Azure account administrator role, 

Keep an updated inventory of the service accounts, which are another primary user type in AKS. A service account is managed by the Kubernetes API. The credentials for service accounts are stored as Kubernetes secrets, which allows them to be used by authorized pods to communicate with the API Server. 

- [Use Azure CLI for operations](https://docs.microsoft.com/cli/azure/aks?view=azure-cli-latest)

- [Understand AKS and Azure AD integration](concepts-identity.md)

- [How to integrate AKS with Azure AD](/azure/aks/azure-ad-integration)

- [Integrate AKS-managed Azure AD (Preview)](managed-aad.md)

- [How to get a directory role in Azure AD with PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0)

- [How to get members of a directory role in Azure AD with PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0)

- [ How to monitor identity and access with Azure Security Center](../security-center/security-center-identity-access.md)

**Azure Security Center monitoring**: Yes

**Responsibility**: Customer

### 3.2: Change default passwords where applicable

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32899.).

**Guidance**: Kubernetes itself does not have the concept of common default passwords or provide an identity management solution where regular user accounts and passwords are stored. Instead, external identity solutions, such as Azure Active Directory (Azure AD) can be integrated into Kubernetes. 

- [Understand access and identity options for AKS](concepts-identity.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 3.3: Use dedicated administrative accounts

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32900.).

**Guidance**: Integrate user authentication for your Azure Kubernetes Service (AKS) clusters with Azure Active Directory (Azure AD). 

Sign in to an AKS cluster using an Azure AD authentication token. Configure Kubernetes role-based access control (RBAC) to limit access to cluster resources based a user's identity or group membership.

Create policies and procedures around the use of dedicated administrative accounts. 

Use Azure AD group membership to control access to namespaces and cluster resources using Kubernetes RBAC in an AKS cluster

Interact with Kubernetes clusters using the kubectl tool. The Azure CLI provides an easy way to get the access credentials and configuration information to connect to your AKS clusters using kubectl. 

Limit access to Kubernetes configuration (kubeconfig) information and permissions with Azure role-based access controls (RBAC).

Implement Security Center Identity and Access Management recommendations.

- [Understand AKS Azure Active Directory integration](concepts-identity.md)

- [How to monitor identity and access with Azure Security Center](../security-center/security-center-identity-access.md)

- [Control access to cluster resources](azure-ad-rbac.md)

- [Use Azure role-based access controls](control-kubeconfig-access.md)

**Azure Security Center monitoring**: Yes

**Responsibility**: Customer

### 3.4: Use single sign-on (SSO) with Azure Active Directory

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32901.).

**Guidance**: Use single sign-on for Azure Kubernetes Service (AKS) as it supports Azure Active Directory authentication for an AKS cluster.

- [How to view Kubernetes logs, events, and pod metrics in real-time](../azure-monitor/insights/container-insights-livedata-overview.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 3.5: Use multi-factor authentication for all Azure Active Directory based access

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32902.).

**Guidance**: Integrate Authentication for Azure Kubernetes Service (AKS) with Azure Active Directory (Azure AD). 

Enable Azure AD multi-factor authentication (MFA) and follow Security Center Identity and Access Management recommendations.

- [How to enable MFA in Azure](../active-directory/authentication/howto-mfa-getstarted.md)

How to monitor identity and access within Azure Security 

**Azure Security Center monitoring**: Yes

**Responsibility**: Customer

### 3.6: Use dedicated machines (Privileged Access Workstations) for all administrative tasks

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32903.).

**Guidance**: Use PAWs (privileged access workstations), with multi-factor authentication (MFA), configured to log into your specified Azure Kubernetes Service (AKS) clusters and related resources.
- [Learn about Privileged Access Workstations](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

- [How to enable MFA in Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Control access to cluster resources using role-based access control and Azure Active Directory identities in Azure Kubernetes Service](azure-ad-rbac.md)

- [Use Azure role-based access controls to define access to the Kubernetes configuration file in Azure Kubernetes Service (AKS)](control-kubeconfig-access.md)

- [Secure access to the API server using authorized IP address ranges in Azure Kubernetes Service (AKS)](api-server-authorized-ip-ranges.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 3.7: Log and alert on suspicious activities from administrative accounts

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32904.).

**Guidance**: Use Azure Active Directory (Azure AD) security reports with Azure AD integrated authentication for Azure Kubernetes Service (AKS). Generate alerts when suspicious or unsafe activity occurs in the environment. 

Use Security Center to monitor identity and access activity.

- [How to identify Azure AD users flagged for risky activity](/azure/active-directory/reports-monitoring/concept-user-at-risk)

- [How to monitor users identity and access activity in Azure Security Center](../security-center/security-center-identity-access.md)

**Azure Security Center monitoring**: Yes

**Responsibility**: Customer

### 3.8: Manage Azure resources only from approved locations

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32905.).

**Guidance**: Use Conditional Access Named Locations to allow access to AKS clusters from only specific logical groupings of IP address ranges or countries/regions. This requires integrated authentication for Azure Kubernetes Service (AKS) with Azure Active Directory (Azure AD).

Limit the access to the AKS API server from a limited set of IP address ranges, as it receives requests to perform actions in the cluster to create resources or scale the number of nodes. 

- [Secure access to the API server using authorized IP address ranges in Azure Kubernetes Service (AKS)](api-server-authorized-ip-ranges.md)

- [How to configure Named Locations in Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 3.9: Use Azure Active Directory

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32906.).

**Guidance**: Use Azure Active Directory (Azure AD) as the central authentication and authorization system for Azure Kubernetes Service (AKS). 

Azure AD protects data by using strong encryption for data at rest and in transit and salts, hashes, and securely stores user credentials.

Use the AKS built-in roles with Azure role-based access control (Azure RBAC)  - Resource Policy Contributor and Owner, for policy assignment operations to your Kubernetes cluster

What is Azure Policy? ../governance/policy/overview.md

For additional reference, AKS uses several managed identities for built-in services and add-ons. use-managed-identity.md#summary-of-managed-identities

- [How to create and configure an Azure AD instance](../active-directory-domain-services/tutorial-create-instance.md)

- [How to integrate Azure AD with AKS](/azure/aks/azure-ad-integration) 

- [Integrate AKS-managed Azure AD (Preview)](managed-aad.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 3.10: Regularly review and reconcile user access

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32907.).

**Guidance**: Use Azure Active Directory (Azure AD) security reports with Azure AD integrated authentication for Azure Kubernetes Service (AKS).

Search Azure AD logs to help discover stale accounts. 

Perform Azure Identity Access Reviews to efficiently manage group memberships, access to enterprise applications, and role assignments. 

Remediate Identity and Access recommendations from Security Center.

Be aware of roles used for support or troubleshooting purposes. 
For example, any cluster actions taken by Microsoft support (with user consent) are made under a built-in Kubernetes "edit" role of the name aks-support-rolebinding. AKS support is enabled with this role to edit cluster configuration and resources to troubleshoot and diagnose cluster issues. However, this role can not modify permissions nor create roles or role bindings. This role access is only enabled under active support tickets with just-in-time (JIT) access.

 
- [Access and identity options for Azure Kubernetes Service (AKS)](concepts-identity.md)

- [Understand Azure AD Logs](../active-directory/reports-monitoring/concept-audit-logs.md)

- [How to use Azure Identity Access Reviews](../active-directory/governance/access-reviews-overview.md)

- [How to monitor user’s identity and access activity in Azure Security Center](../security-center/security-center-identity-access.md)

**Azure Security Center monitoring**: Yes

**Responsibility**: Customer

### 3.11: Monitor attempts to access deactivated credentials

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32908.).

**Guidance**: Integrate user authentication for Azure Kubernetes Service (AKS) with Azure Active Directory. Create Diagnostic Settings for Azure Active Directory, sending the audit and sign-in logs to an Azure Log Analytics workspace. Configure desired Alerts (such as when a deceived account attempts to login) within Azure Log Analytics workspace.
- [How to integrate Azure Activity Logs into Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

- [How to create, view, and manage log alerts using Azure Monitor](../azure-monitor/platform/alerts-log.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 3.12: Alert on account login behavior deviation

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32909.).

**Guidance**: Integrate user authentication for Azure Kubernetes Service (AKS) with Azure Active Directory (Azure AD). 

Use Azure AD Risk Detections and Identity Protection feature to configure automated responses to detected suspicious actions related to user identities. 

Ingest data into Azure Sentinel for further investigations, as required based on your business needs.

- [How to view Azure AD risky sign-ins](/azure/active-directory/reports-monitoring/concept-risky-sign-ins)

- [How to configure and enable Identity Protection risk policies](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [How to onboard Azure Sentinel](../sentinel/quickstart-onboard.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 3.13: Provide Microsoft with access to relevant customer data during support scenarios

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32910.).

**Guidance**: Not available to Azure Kubernetes Service (AKS). Customer Lockbox is not yet supported AKS.

- [List of Customer Lockbox supported services](../security/fundamentals/customer-lockbox-overview.md#supported-services-and-scenarios-in-general-availability)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

## Data protection

*For more information, see the [Azure Security Benchmark: Data protection](/azure/security/benchmarks/security-control-data-protection).*

### 4.1: Maintain an inventory of sensitive Information

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32911.).

**Guidance**: Use tags on resources related to Azure Kubernetes Service (AKS) deployments to assist in tracking Azure resources that store or process sensitive information.

- [How to create and use Tags](/azure/azure-resource-manager/resource-group-using-tags)

- [Managed Clusters - update tags](/rest/api/aks/managedclusters/updatetags)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 4.2: Isolate systems storing or processing sensitive information

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32912.).

**Guidance**: Logically isolate teams and workloads in the same cluster with Azure Kubernetes Service (AKS) to provide the least number of privileges, scoped to the resources required by each team. 

Use the namespace in Kubernetes to create a logical isolation boundary. Consider implementing additional Kubernetes features and considerations for isolation and multi-tenancy, such as, scheduling, networking, authentication/authorization, and containers.

Implement separate subscriptions and/or management groups for development, test, and production environments. 

Separate AKS clusters with virtual network/subnet, tagged appropriately, and secured within a Web Application Firewall.

- [Learn about best practices for cluster isolation in AKS](operator-best-practices-cluster-isolation.md)

- [How to create additional Azure subscriptions](/azure/billing/billing-create-subscription)

- [How to create Management Groups](../governance/management-groups/create.md)

- [Understand best practices for network connectivity and security in AKS](operator-best-practices-network.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 4.3: Monitor and block unauthorized transfer of sensitive information

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32913.).

**Guidance**: Restrict egress traffic to addresses, ports, and domain names that as required to increase the security of your Azure Kubernetes Service (AKS) cluster, 

Use Azure Firewall or a third-party firewall appliance to secure egress traffic and define the required addresses, ports, and domain names. Note that AKS does not automatically create these rules for you.

Enable Diagnostic Settings within the Azure Firewall to log rule matches. 

Use Azure Monitor Activity Log to detect changes to the Azure Firewalls being used to limit egress traffic on your AKS clusters.

For the underlying platform which is managed by Microsoft, Microsoft treats all customer content as sensitive and goes to great lengths to guard against customer data loss and exposure. To ensure customer data within Azure remains secure, Microsoft has implemented and maintains a suite of robust data protection controls and capabilities.

- [List of required ports, addresses, and domain names for AKS functionality](limit-egress-traffic.md)

- [How to configure Diagnostic Settings for Azure Firewall](/azure/firewall/tutorial-diagnostics)

- [How to create alerts in Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

- [Understand customer data protection in Azure](../security/fundamentals/protection-customer-data.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Microsoft

### 4.4: Encrypt all sensitive information in transit

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32914.).

**Guidance**: Create an HTTPS ingress controller and use your own TLS certificates (or optionally, Let's Encrypt) for your Azure Kubernetes Service (AKS) deployments.

Kubernetes egress traffic is encrypted by default over HTTPS/TLS. For example, the communication with the API server or downloading core Kubernetes cluster components and node security updates is encrypted by default.

Note that there may be potentially un-encrypted egress traffic from your AKS instances which includes NTP traffic, DNS traffic, HTTP traffic for retrieving updates in some cases.

- [How to create an HTTPS ingress controller on AKS and use your own TLS certificates](ingress-own-tls.md)

- [How to create an HTTPS ingress controller on AKS with Let's Encrypt](ingress-tls.md)

- [List of potential out-going ports and protocols used by AKS](limit-egress-traffic.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 4.5: Use an active discovery tool to identify sensitive data

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32915.).

**Guidance**: Data identification, classification, and loss prevention features are not yet available for Azure Storage or compute resources. Implement third-party solution if required for compliance purposes.
For the underlying platform which is managed by Microsoft, Microsoft treats all customer content as sensitive and goes to great lengths to guard against customer data loss and exposure. 

To ensure customer data within Azure remains secure, Microsoft has implemented and maintains a suite of robust data protection controls and capabilities.

- [Understand customer data protection in Azure](../security/fundamentals/protection-customer-data.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 4.6: Use Azure RBAC to manage access to resources

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32916.).

**Guidance**: Use the Azure role-based access control (Azure RBAC) authorization system built on Azure Resource Manager to provide fine-grained access management of Azure resources.

Create a role definition using Azure RBAC that outlines the permissions to be applied. A user or group can be assigned this role definition via a role assignment for a particular scope, such as an individual resource, a resource group, or across the subscription.

Configure Azure Kubernetes Service (AKS) to use Azure Active Directory (Azure AD) for user authentication. Sign in to an AKS cluster using an Azure Active Directory authentication token using this configuration. 

- [How to control access to cluster resources using Azure RBAC and Azure AD Identities in AKS](azure-ad-rbac.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 4.7: Use host-based data loss prevention to enforce access control

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32917.).

**Guidance**: Data identification, classification, and loss prevention features are not yet available for Azure Storage or compute resources. Implement third-party solution if required for compliance purposes.
For the underlying platform which is managed by Microsoft, Microsoft treats all customer content as sensitive and goes to great lengths to guard against customer data loss and exposure. To ensure customer data within Azure remains secure, Microsoft has implemented and maintains a suite of robust data protection controls and capabilities.

- [Understand customer data protection in Azure](../security/fundamentals/protection-customer-data.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 4.8: Encrypt sensitive information at rest

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32918.).

**Guidance**: Azure Storage encrypts all data in a storage account at rest. By default, data is encrypted with Microsoft-managed keys. 

Supply customer-managed keys (BYOK) to use for encryption of both the OS and data disks for your Azure Kubernetes Service (AKS) clusters for additional control over encryption keys.

Customers own the responsibility for key management activities such as key backup and rotation.

- [Understand encryption-at-rest and BYOK with AKS](azure-disk-customer-managed-keys.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 4.9: Log and alert on changes to critical Azure resources

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32919.).

**Guidance**: Use Azure Monitor for containers to monitor the performance of container workloads deployed to managed Kubernetes clusters hosted on Azure Kubernetes Service (AKS). 

Configure alerts to proactive notification or log creation when CPU and memory utilization on nodes or containers exceed defined thresholds, or when a health state change occurs in the cluster at the infrastructure or nodes health rollup. 

Use Azure Activity Log to monitor your AKS clusters and related resources at a high level. 

Integrate with Prometheus to view application and workload metrics it collects from nodes and Kubernetes using queries to create custom alerts, dashboards, and detailed perform detailed analysis.

- [Understand Azure Monitor for Containers](../azure-monitor/insights/container-insights-overview.md)

- [How to enable Azure Monitor for containers](../azure-monitor/insights/container-insights-onboard.md)

- [How to view and retrieve Azure Activity Log events](/azure/azure-monitor/platform/activity-log-view)

**Azure Security Center monitoring**: Yes

**Responsibility**: Customer

## Vulnerability management

*For more information, see the [Azure Security Benchmark: Vulnerability management](/azure/security/benchmarks/security-control-vulnerability-management).*

### 5.1: Run automated vulnerability scanning tools

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32920.).

**Guidance**: Use Security Center to monitor your Azure Container Registry including Azure Kubernetes Service (AKS) instances for vulnerabilities.

Enable the optional Container Registries bundle in Security Center to ensure that Security Center is ready to scan images that get pushed to the registry.

Get notified in the Security Center dashboard when issues are found after Security Center scans the image using Qualys. This optional Container Registries bundle feature provides deeper visibility into  vulnerabilities of the images used in Azure Resource Manager (ARM) based registries. 
You can choose to enable or disable the bundle at the subscription level to cover all registries in a subscription. 

Use Security Center for actionable recommendations for every vulnerability. These recommendations include a severity classification and guidance for how to remediate the issue. 

- [Best practices for container image management and security in Azure Kubernetes Service (AKS)](../security-center/azure-container-registry-integration.md)

- [Understand best practices for container image management and security in AKS](operator-best-practices-container-image-management.md)

- [Understand container Registry integration with Azure Security Center](../security-center/azure-container-registry-integration.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 5.2: Deploy automated operating system patch management solution

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32921.).

**Guidance**: Security updates are automatically applied to Linux nodes to protect customer's Azure Kubernetes Service (AKS) clusters. These updates include OS security fixes or kernel updates. 

Note that the process to keep Windows Server nodes up to date differs from nodes running Linux as windows Server nodes don't receive daily updates. 

Instead, an AKS upgrade is required by the customer which deploys new nodes with the latest base Window Server image and patches. 

Customers are responsible for executing Kubernetes upgrades. 
They can execute upgrades through the Azure control panel or the Azure CLI. This includes updates that contain security or functionality improvements to Kubernetes.

- [Understand how updates are applied to AKS cluster nodes running Linux](node-updates-kured.md)

- [How to upgrade an AKS node pool for AKS clusters that use Windows Server nodes](use-multiple-node-pools.md#upgrade-a-node-pool)

- [Azure Kubernetes Service (AKS) node image upgrades](node-image-upgrade.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 5.3: Deploy an automated patch management solution for third-party software titles

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32922.).

**Guidance**: Implement a manual process to ensure Azure Kubernetes Service (AKS) cluster node's third-party applications remain patched for the duration of the cluster lifetime which may require enabling automatic updates, monitoring the nodes, or performing periodic reboots.

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 5.4: Compare back-to-back vulnerability scans

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32923.).

**Guidance**: Export Security Center scan results at consistent intervals and compare the results to verify that vulnerabilities have been remediated. 

Use the PowerShell cmdlet "Get-AzSecurityTask" to automate the retrieval of security tasks that Security Center recommends you to perform in order to strengthen your security posture and remediation vulnerability scan findings.

- [How to use PowerShell to view vulnerabilities discovered by Azure Security Center](https://docs.microsoft.com/powershell/module/az.security/get-azsecuritytask?view=azps-3.3.0)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 5.5: Use a risk-rating process to prioritize the remediation of discovered vulnerabilities

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32924.).

**Guidance**: Use the severity rating provided by Security Center to prioritize the remediation of vulnerabilities. 

Use Common Vulnerability Scoring System (CVSS) (or another scoring systems as provided by your scanning tool) if using a built-in vulnerability assessment tool (such as Qualys or Rapid7, offered by Azure).

**Azure Security Center monitoring**: Yes

**Responsibility**: Customer

## Inventory and asset management

*For more information, see the [Azure Security Benchmark: Inventory and asset management](/azure/security/benchmarks/security-control-inventory-asset-management).*

### 6.1: Use automated asset discovery solution

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32925.).

**Guidance**: Use Azure Resource Graph to query/discover all resources (such as compute, storage, network, and so on) within your subscriptions. 

Ensure that you have appropriate (read) permissions in your tenant and are able to enumerate all Azure subscriptions as well as resources within your subscriptions.

Although classic Azure resources may be discovered via Resource Graph, it is highly recommended to create and use Azure Resource Manager based resources going forward.

- [How to create queries with Azure Graph](../governance/resource-graph/first-query-portal.md)

- [How to view your Azure Subscriptions](https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0)

- [Understand Azure RBAC](../role-based-access-control/overview.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 6.2: Maintain asset metadata

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32926.).

**Guidance**: Apply tags to Azure resources with metadata to logically organize them into a taxonomy.

- [How to create and use tags](/azure/azure-resource-manager/resource-group-using-tags)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 6.3: Delete unauthorized Azure resources

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32927.).

**Guidance**: Use tagging, management groups, and separate subscriptions, where appropriate, to organize and track assets. 

Add taints, labels, or tags when creating an Azure Kubernetes Service (AKS) node pool. All nodes within that node pool will also inherit that taint, label, or tag.

Use taints, labels and/or tags to reconcile inventory on a regular basis and ensure unauthorized resources are deleted from subscriptions in a timely manner.

- [How to create additional Azure subscriptions](/azure/billing/billing-create-subscription)

- [How to create Management Groups](../governance/management-groups/create.md)

- [How to create and user Tags](/azure/azure-resource-manager/resource-group-using-tags)

- [Managed Clusters - Update Tags](/rest/api/aks/managedclusters/updatetags)

- [Specify a taint, label, or tag for a node pool](use-multiple-node-pools.md#specify-a-taint-label-or-tag-for-a-node-pool)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 6.4: Define and maintain an inventory of approved Azure resources

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32928.).

**Guidance**: Define a list of approved Azure resources and approved software for compute resources based on organizational business needs.

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 6.5: Monitor for unapproved Azure resources

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32929.).

**Guidance**: Use Azure Policy to put restrictions on the type of resources that can be created in customer subscriptions using the following built-in policy definitions:
-	Not allowed resource types 

-	Allowed resource types

Use Azure Resource Graph to query/discover resources within your subscriptions. Ensure that all Azure resources present in the environment are approved based on organizational business requirements.

- [How to configure and manage Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [How to create queries with Azure Graph](../governance/resource-graph/first-query-portal.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 6.6: Monitor for unapproved software applications within compute resources

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32930.).

**Guidance**: Use Azure Automation Change Tracking and Inventory features to find out software that is installed in your environment. 

Collect and view inventory for software, files, Linux daemons, Windows services, and Windows Registry keys on your computers and monitor for unapproved software applications. 

Track the configurations of your machines to aid in pinpointing operational issues across your environment and better understand the state of your machines.

- [How to enable Azure virtual machine Inventory](../automation/automation-tutorial-installed-software.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 6.7: Remove unapproved Azure resources and software applications

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32931.).

**Guidance**: Use Azure Automation Change Tracking and Inventory features to find out software that is installed in your environment. 

Collect and view inventory for software, files, Linux daemons, Windows services, and Windows Registry keys on your computers and monitor for unapproved software applications. 

Track the configurations of your machines to aid in pinpointing operational issues across your environment and better understand the state of your machines.

- [How to enable Azure virtual machine Inventory](../automation/automation-tutorial-installed-software.md)

- [How to use File Integrity Monitoring](../security-center/security-center-file-integrity-monitoring.md#using-file-integrity-monitoring)

- [Understand Azure Change Tracking](../automation/change-tracking.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 6.8: Use only approved applications

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32932.).

**Guidance**: Use Azure Automation Change Tracking and Inventory features to find out software that is installed in your environment. 

Collect and view inventory for software, files, Linux daemons, Windows services, and Windows Registry keys on your computers and monitor for unapproved software applications. 

Track the configurations of your machines to aid in pinpointing operational issues across your environment and better understand the state of your machines.

Enable Adaptive Application analysis in Security Center for applications which exist in your environment.

- [How to enable Azure virtual machine Inventory](../automation/automation-tutorial-installed-software.md)

 How
to use Azure Security Center Adaptive Application
- [Controls](../security-center/security-center-adaptive-application.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 6.9: Use only approved Azure services

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32933.).

**Guidance**: Use Azure Policy to put restrictions on the type of resources that can be created in customer subscriptions using the following built-in policy definitions:

- Not allowed resource types

- Allowed resource types

Use Azure Resource Graph to query/discover resources within your subscriptions. Ensure that all Azure resources present in the environment are approved.

- [How to configure and manage Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [How to configure and manage Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [How to deny a specific resource type with Azure Policy](/azure/governance/policy/samples/not-allowed-resource-types)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 6.10: Maintain an inventory of approved software titles

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32934.).

**Guidance**: Use Azure Policy to put restrictions on the type of resources that can be created in your subscriptions using built-in policy definitions.

- [How to configure and manage Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 6.11: Limit users' ability to interact with Azure Resource Manager

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32935.).

**Guidance**: Use Azure Conditional Access to limit users' ability to interact with Azure Resource Manager (ARM) by configuring "Block access" for the "Microsoft Azure Management" App.
- [How to configure Conditional Access to block access to ARM](../role-based-access-control/conditional-access-azure-management.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 6.12: Limit users' ability to execute scripts in compute resources

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32936.).

**Guidance**: Azure Kubernetes Service (AKS) itself doesn't provide an identity management solution where regular user accounts and passwords are stored. Instead, use Azure Active Directory (Azure AD) as the integrated identity solution for your AKS clusters. 

Grant users or groups access to Kubernetes resources within a namespace or across the cluster using Azure AD integration. 

Use the Azure AD PowerShell module to perform ad hoc queries to discover accounts that are members of your AKS administrative groups; reconcile access on a regular basis. 

Use Azure CLI for operations such as ‘Get access credentials for a managed Kubernetes cluster.

Implement Security Center Identity and Access Management recommendations.

- [Manage AKS with Azure CLI](https://docs.microsoft.com/cli/azure/aks?view=azure-cli-latest)

- [Understand AKS and Azure AD integration](concepts-identity.md)

- [How to integrate AKS with Azure AD](/azure/aks/azure-ad-integration)

- [Integrate AKS-managed Azure AD (Preview)](managed-aad.md)

- [How to get a directory role in Azure AD with PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0)

- [How to get members of a directory role in Azure AD with PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0)

- [How to monitor identity and access with Azure Security Center](../security-center/security-center-identity-access.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 6.13: Physically or logically segregate high risk applications

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32937.).

**Guidance**: Use Azure Kubernetes Service (AKS) features to logically isolate teams and workloads in the same cluster. This provides the least number of privileges, scoped to the resources each team needs. 

Implement namespace in Kubernetes to create a logical isolation boundary. 

Review and implement additional Kubernetes features and considerations for isolation and multi-tenancy include the following areas: scheduling, networking, authentication/authorization, and containers.

Also use separate subscriptions and/or management groups for development, test, and production. 

Separate AKS clusters with virtual networks, subnets which are tagged appropriately, and secured with a Web Application Firewall (WAF).

- [Learn about best practices for cluster isolation in AKS](operator-best-practices-cluster-isolation.md)

- [How to create additional Azure subscriptions](/azure/billing/billing-create-subscription)

- [How to create Management Groups](../governance/management-groups/create.md)

- [Understand best practices for network connectivity and security in AKS](operator-best-practices-network.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

## Secure configuration

*For more information, see the [Azure Security Benchmark: Secure configuration](/azure/security/benchmarks/security-control-secure-configuration).*

### 7.1: Establish secure configurations for all Azure resources

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32938.).

**Guidance**: Use Azure Policy aliases in the "Microsoft.ContainerService" namespace to create custom policies to audit or enforce the configuration of your Azure Kubernetes Service (AKS) instances. 

Use Azure Policy to put restrictions on the type of resources that can be created in your subscription(s) using built-in policy definitions.

- [How to configure and manage AKS pod security policies](use-pod-security-policies.md)

- [How to configure and manage Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 7.2: Establish secure operating system configurations

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32939.).

**Guidance**: Azure Kubernetes Service (AKS) is a secure service which is already compliant with SOC, ISO, PCI DSS, and HIPAA standards. 

AKS clusters are deployed on host virtual machines with a security optimized OS. The host OS has additional security hardening steps incorporated into it to reduce the surface area of attack and allows the deployment of containers in a secure fashion. 

Azure applies daily patches (including security patches) to AKS virtual machine hosts with some patches requiring a reboot. 

Customers are responsible for scheduling AKS VM host reboots as needed. 

- [Security hardening for AKS agent node host OS](security-hardened-vm-host-image.md)

- [Understand security hardening in AKS virtual machine hosts](security-hardened-vm-host-image.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 7.3: Maintain secure Azure resource configurations

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32940.).

**Guidance**: Secure your Azure Kubernetes Service (AKS) cluster using pod security policies.  Limit what pods can be scheduled to improve the security of your cluster. Pods which request resources that are not allowed cannot run in the AKS cluster. Define this access using pod security policies.

Also use Azure Policy [deny] and [deploy if not exist] to enforce secure settings for the Azure resources related to your AKS deployments (such as Virtual Networks, Subnets, Azure Firewalls, Storage Accounts, and so on). 

Implement Azure Policy Aliases from the following namespaces to create custom policies:

•	Microsoft.ContainerService

•	Microsoft.Network

Use Azure Policy to put restrictions on the type of resources that can be created in your subscription(s) using built-in policy definitions.

- [Understand Azure Policy for Kubernetes clusters (preview)](../governance/policy/concepts/policy-for-kubernetes.md)

- [How to configure and manage Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Understand Azure Policy Effects](../governance/policy/concepts/effects.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 7.4: Maintain secure operating system configurations

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32941.).

**Guidance**: Azure Kubernetes Service (AKS) clusters are deployed on host virtual machines with a security optimized OS. The host OS has additional security hardening steps incorporated into it to reduce the surface area of attack and allows the deployment of containers in a secure fashion. 

Azure applies daily patches (including security patches) to AKS virtual machine hosts with some patches requiring a reboot. 

Customers are responsible for scheduling AKS Virtual Machine host reboots as per need. 

- [Security hardening for AKS agent node host OS](security-hardened-vm-host-image.md)

- [Understand security hardening in AKS virtual machine hosts](security-hardened-vm-host-image.md)

- [Understand state configuration of AKS clusters](concepts-clusters-workloads.md#control-plane)

- [Understand security hardening in AKS virtual machine hosts](security-hardened-vm-host-image.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 7.5: Securely store configuration of Azure resources

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32942.).

**Guidance**: Use Azure DevOps repos to securely store and manage your configurations if using custom Azure policy definitions.

Security hardening for AKS agent node host OS

security-hardened-vm-host-image.md

- [How to store code in Azure DevOps](https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops)

- [Azure Repos Documentation](https://docs.microsoft.com/azure/devops/repos/index?view=azure-devops)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 7.6: Securely store custom operating system images

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32943.).

**Guidance**: Not available to Azure Kubernetes Service (AKS). AKS provides a security optimized host Operating System (OS) by default. There is no current option to select an alternate or custom operating system.

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 7.7: Deploy configuration management tools for Azure resources

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32944.).

**Guidance**: Use Azure Policy to put restrictions on the type of resources that can be created in subscriptions using built-in policy definitions as well as Azure Policy aliases in the "Microsoft.ContainerService" namespace. 

Create custom policies to alert, audit, and enforce system configurations. 

Develop a process and pipeline for managing policy exceptions.

- [How to configure and manage Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [How to use aliases](../governance/policy/concepts/definition-structure.md#aliases)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 7.8: Deploy configuration management tools for operating systems

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32945.).

**Guidance**: Azure Kubernetes Service (AKS) clusters are deployed on host virtual machines with a security optimized Operating System (OS). The host OS has additional security hardening steps incorporated into it to reduce the surface area of attack and allows the deployment of containers in a secure fashion. 

Azure applies daily patches (including security patches) to AKS virtual machine hosts with some patches requiring a reboot. 

Customers are responsible for scheduling AKS Virtual Machine host reboots as per need. 

- [Security hardening for AKS agent node host OS](security-hardened-vm-host-image.md)

- [Understand security hardening in AKS virtual machine hosts](security-hardened-vm-host-image.md)

- [Understand state configuration of AKS clusters](concepts-clusters-workloads.md#control-plane)

- [Understand security hardening in AKS virtual machine hosts](security-hardened-vm-host-image.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 7.9: Implement automated configuration monitoring for Azure resources

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32946.).

**Guidance**: Use Security Center to perform baseline scans for resources related to your Azure Kubernetes Service (AKS) deployments. Examples resources include but are not limited to the AKS cluster itself, the virtual network where the AKS cluster was deployed, the Azure Storage Account used to track Terraform state, or Azure Key Vault instances being used for the encryption keys for your AKS cluster's Operating System (OS) and data disks.

- [How to remediate recommendations in Azure Security Center](../security-center/security-center-remediate-recommendations.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 7.10: Implement automated configuration monitoring for operating systems

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32947.).

**Guidance**: Use Security Center container recommendations under the "Compute &amp; apps" section to perform baseline scans for your Azure Kubernetes Service (AKS) clusters. 
Get notified in the Security Center dashboard when configuration issues or vulnerabilities are found. This does require enabling the optional Container Registries bundle which allows Security Center to scan the image.  

- [Understand Azure Security Center container recommendations](/azure/security-center/security-center-container-recommendations)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 7.11: Manage Azure secrets securely

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32948.).

**Guidance**: Use Azure Key Vault to store and regularly rotate secrets such as credentials, storage account keys, or certificates. 

Integrate Azure Key Vault with an Azure Kubernetes Service (AKS) cluster using a FlexVolume drive. The FlexVolume driver lets the AKS cluster natively retrieve credentials from Key Vault and securely provide them only to the requesting pod. 

Use a pod managed identity to request access to Key Vault and retrieve the required credentials through the FlexVolume driver. Ensure Key Vault Soft Delete is enabled. 

Limit credential exposure by not defining credentials in your application code. 

Avoid the use of fixed or shared credentials. 

- [Security concepts for applications and clusters in Azure Kubernetes Service (AKS)](concepts-security.md)

- [How to use Key Vault with your AKS cluster](developer-best-practices-pod-security.md#limit-credential-exposure)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 7.12: Manage identities securely and automatically

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32949.).

**Guidance**: Do not define credentials in your application code as a security best practice.

Use managed identities for Azure resources lets a pod authenticate itself against any service in Azure that supports it including  Azure Key Vault. 
The pod is assigned an Azure Identity to authenticate to Azure Active Directory (Azure AD) and receive a digital token. This digital token can be presented to other Azure services that check if the pod is authorized to access the service and perform the required actions. Note that Pod managed identities is intended for use with Linux pods and container images only.

Provision Azure Key Vault to store and retrieve digital keys and credentials. 

Service principals can also be used in AKS clusters. However, clusters using service principals eventually may reach a state in which the service principal must be renewed to keep the cluster working. Managing service principals adds complexity, which is why it's easier to use managed identities instead. The same permission requirements apply for both service principals and managed identities.

- [Understand Managed Identities and Key Vault with Azure Kubernetes Service (AKS)](developer-best-practices-pod-security.md#limit-credential-exposure)

- [Azure Active Directory Pod Identity](https://github.com/Azure/aad-pod-identity)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 7.13: Eliminate unintended credential exposure

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32950.).

**Guidance**: Implement Credential Scanner to identify credentials within code. Credential Scanner also encourages moving discovered credentials to more secure locations such as Azure Key Vault with recommendations.

Limit credential exposure by not defining credentials in your application code. and avoid the use of shared credentials. 

Azure Key Vault should be used to store and retrieve digital keys and credentials. 

Use managed identities for Azure resources to let your pod request access to other resources. 

- [How to setup Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

- [Developer best practices for pod security](developer-best-practices-pod-security.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

## Malware defense

*For more information, see the [Azure Security Benchmark: Malware defense](/azure/security/benchmarks/security-control-malware-defense).*

### 8.1: Use centrally managed antimalware software

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32951.).

**Guidance**: Not applicable to Azure Kubernetes Service (AKS). This recommendation is intended for platform resources.

- [How to deploy Microsoft Antimalware](../security/fundamentals/antimalware.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 8.2: Pre-scan files to be uploaded to non-compute Azure resources

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32952.).

**Guidance**: Microsoft Antimalware is enabled on the underlying host that supports Azure Azure Kubernetes Service (AKS). However, it does not run on the Linux compute resources in your subscription.

Pre-scan any files or applications being uploaded to your Linux compute or related resources.

Use Security Center's threat detection for data services to detect malware uploaded to storage accounts if using an Azure Storage Account as a data store or to track Terraform state for your AKS cluster. 

- [Understand Microsoft Antimalware for Azure Cloud Services and Virtual Machines](../security/fundamentals/antimalware.md)

- [Understand Azure Security Center's Threat detection for data services](/azure/security-center/security-center-alerts-data-services)

**Azure Security Center monitoring**: Yes

**Responsibility**: Customer

### 8.3: Ensure antimalware software and signatures are updated

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32953.).

**Guidance**: Microsoft Antimalware automatically installs the latest signatures and engine updates by default for Windows Server Azure Kubernetes Service (AKS) cluster nodes.

Follow recommendations in Azure Security Center under the "Compute &amp; Apps" section to confirm all endpoints are up to date with the latest signatures. 

Deploy your own anti-malware solution for Linux nodes and ensure the engine and signatures remain up-to-date.

- [How to deploy Microsoft Antimalware for Azure Cloud Services and Virtual Machines](../security/fundamentals/antimalware.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

## Data recovery

*For more information, see the [Azure Security Benchmark: Data recovery](/azure/security/benchmarks/security-control-data-recovery).*

### 9.1: Ensure regular automated back ups

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32954.).

**Guidance**: Back up your data using an appropriate tool for your storage type such as Velero. Velero can back up persistent volumes along with additional cluster resources and configurations. 

Periodically, verify the integrity, and security, of those backups. 

Remove state from your applications prior to backup. In cases where this cannot be done, back up the data from persistent volumes and regularly test the restore operations to verify data integrity and the processes required.

- [Best practices for storage and backups in AKS](operator-best-practices-storage.md)

- [Best practices for business continuity and disaster recovery in AKS](operator-best-practices-multi-region.md)

- [Understand Azure Site Recovery](../site-recovery/site-recovery-overview.md)

- [How to setup Velero on Azure](https://github.com/vmware-tanzu/velero-plugin-for-microsoft-azure/blob/master/README.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 9.2: Perform complete system backups and backup any customer-managed keys

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32955.).

**Guidance**: Back up your data using an appropriate tool for your storage type such as Velero. Velero can back up persistent volumes along with additional cluster resources and configurations. 

Perform regular automated backups of Key Vault Certificates, Keys, Managed Storage Accounts, and Secrets, with PowerShell commands. 

For example:

Backup-AzKeyVaultCertificate Backup-AzKeyVaultKey Backup-AzKeyVaultManagedStorageAccount Backup-AzKeyVaultSecret

- [How to backup Key Vault Certificates](/powershell/module/azurerm.keyvault/backup-azurekeyvaultcertificate)

- [How to backup Key Vault Keys](/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey)

- [How to backup Key Vault Managed Storage Accounts](/powershell/module/az.keyvault/add-azkeyvaultmanagedstorageaccount)

- [How to backup Key Vault Secrets](/powershell/module/azurerm.keyvault/backup-azurekeyvaultsecret)

- [How to enable Azure Backup](/azure/backup)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 9.3: Validate all backups including customer-managed keys

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32956.).

**Guidance**: Periodically perform data restoration of content within Velero Backup. If necessary, test restore to an isolated virtual network.

Periodically perform data restoration of Key Vault Certificates, Keys, Managed Storage Accounts, and Secrets, with PowerShell commands. 
For example:

Restore-AzKeyVaultCertificate Restore-AzKeyVaultKey Restore-AzKeyVaultManagedStorageAccount Restore-AzKeyVaultSecret

- [How to restore Key Vault Certificates](https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultcertificate?view=azurermps-6.13.0)

- [How to restore Key Vault Keys](https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0)

- [How to restore Key Vault Managed Storage Accounts](/powershell/module/az.keyvault/backup-azkeyvaultmanagedstorageaccount)

- [How to restore Key Vault Secrets](https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultsecret?view=azurermps-6.13.0)

- [How to recover files from Azure Virtual Machine backup](/azure/backup/backup-azure-restore-files-from-vm)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

### 9.4: Ensure protection of backups and customer-managed keys

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32957.).

**Guidance**: Back up your data using an appropriate tool for your storage type such as Velero. Velero can back up persistent volumes along with additional cluster resources and configurations. 

Enable Soft-Delete in Key Vault to protect keys against accidental or malicious deletion if Azure Key Vault is being used with for Azure Kubernetes Service (AKS) deployments.

- [Understand Azure Storage Service Encryption](../storage/common/storage-service-encryption.md)

- [How to enable Soft-Delete in Key Vault](https://docs.microsoft.com/azure/storage/blobs/storage-blob-soft-delete?tabs=azure-portal)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Not applicable

## Incident response

*For more information, see the [Azure Security Benchmark: Incident response](/azure/security/benchmarks/security-control-incident-response).*

### 10.1: Create an incident response guide

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32958.).

**Guidance**: Build out an incident response guide for your organization. Ensure that there are written incident response plans that define all roles of personnel as well as phases of incident handling/management from detection to post-incident review.

- [How to configure Workflow Automations within Azure Security Center](../security-center/security-center-planning-and-operations-guide.md)

- [Guidance on building your own security incident response process](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Microsoft Security Response Center's Anatomy of an Incident](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Customer may also leverage NIST's Computer Security Incident Handling Guide to aid in the creation of their own incident response plan](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 10.2: Create an incident scoring and prioritization procedure

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32959.).

**Guidance**: Prioritize which alerts must be investigated first with Security Center assigned severity to alerts. The severity is based on how confident Security Center is in the finding or the analytic used to issue the alert as well as the confidence level that there was malicious intent behind the activity that led to the alert.
Clearly mark subscriptions (for example, production, non-production) and create a naming system to clearly identify and categorize Azure resources.

**Azure Security Center monitoring**: Yes

**Responsibility**: Customer

### 10.3: Test security response procedures

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32964.).

**Guidance**: Conduct exercises to test your systems’ incident response capabilities on a regular cadence. Identify weak points and gaps and revise plan as needed.

Refer to NIST's publication: Guide to Test, Training, and Exercise Programs for IT 
- [Plans and Capabilities](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 10.4: Provide security incident contact details and configure alert notifications for security incidents

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32960.).

**Guidance**: Security incident contact information will be used by Microsoft to contact you if the Microsoft Security Response Center (MSRC) discovers that the customer's data has been accessed by an unlawful or unauthorized party. Review incidents after the fact to ensure that issues are resolved.

- [How to set the Azure Security Center Security Contact](../security-center/security-center-provide-security-contact-details.md)

**Azure Security Center monitoring**: Yes

**Responsibility**: Customer

### 10.5: Incorporate security alerts into your incident response system

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32961.).

**Guidance**: Export Security Center alerts and recommendations using the Continuous Export feature. Continuous Export allows you to export alerts and recommendations either manually or in an ongoing, continuous fashion. 

Or use the Azure Security Center data connector to stream the alerts to Azure Sentinel.

- [How to configure continuous export](../security-center/continuous-export.md)

- [How to stream alerts into Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

### 10.6: Automate the response to security alerts

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32962.).

**Guidance**: Use the Workflow Automation feature in Azure Security Center to automatically trigger responses via "Logic Apps" on security alerts and recommendations.

- [How to configure Workflow Automation and Logic Apps](../security-center/workflow-automation.md)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Customer

## Penetration tests and red team exercises

*For more information, see the [Azure Security Benchmark: Penetration tests and red team exercises](/azure/security/benchmarks/security-control-penetration-tests-red-team-exercises).*

### 11.1: Conduct regular penetration testing of your Azure resources and ensure remediation of all critical security findings

>[!NOTE]
> To revise the text in this section, update the [underlying Work Item](https://dev.azure.com/AzureSecurityControlsBenchmark/AzureSecurityControlsBenchmarkContent/_workitems/edit/32963.).

**Guidance**: Follow the Microsoft Rules of Engagement to ensure your Penetration Tests are not in violation of Microsoft policies:
https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1

- [You can find more information on Microsoft’s strategy and execution of Red Teaming and live site penetration testing against Microsoft-managed cloud infrastructure, services, and applications, here](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Azure Security Center monitoring**: Not applicable

**Responsibility**: Shared

## Next steps

- See the [Azure security benchmark](/azure/security/benchmarks/overview)
- Learn more about [Azure security baselines](/azure/security/benchmarks/security-baselines-overview)
