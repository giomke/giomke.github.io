---
title: "Azure Network Security Monitoring"
date: 2024-1-29T20:00:04+00:00
tags: ["Azure","KQL","Detection","Scan","log","flow","hunting","network","vnet","alert"]
description: "detecting horizontal and vertical scans"
---
## Introduction:
In today’s dynamic cloud environments, detecting and mitigating network scans is crucial for ensuring the security and health of your infrastructure. Azure provides robust tools like VNet Flow Logs, Log Analytics, and custom KQL (Kusto Query Language) rules that can help you detect and respond to both horizontal and vertical network scans. In this post, we will dive into how you can leverage these tools to build effective detection mechanisms for network scans in Azure. We will cover key concepts like VNet flow logs, network architecture best practices, and provide step-by-step guidance on creating custom KQL queries to identify scanning activities in your environment. Whether you’re an Azure security enthusiast or a network administrator, this guide will empower you to enhance your cloud network monitoring.

##  VNet flow logs
![image](/images/1.png)

Virtual network flow logs became generally available 9 months ago. With the help of VNet flow logging, we can now generate NetFlow/IPFIX-style logs. Virtual network flow logs enable the detection of security vulnerabilities and assist in threat-hunting investigations by providing a complete trail of user and application activity. Previously, network security group (NSG) flow logs were used, but they were cumbersome. On September 30, 2027, NSG flow logs will be retired. As part of this retirement, you will no longer be able to create new NSG flow logs starting June 30, 2025.

Using Virtual Network Flow Logs to monitor traffic is a game-changer for network security and performance management. These tools provide invaluable insights into traffic behavior, enabling you to identify patterns and deviations that could indicate potential security compromises. By leveraging these capabilities, you can proactively address threats before they escalate, ensuring your network remains secure.

## Network Architecture
![image](/images/2.png)

Before we dive into the subject, I want to first discuss Hub and Spoke networks to clarify our journey. If this topology is already familiar to you, feel free to skip this section.

A hub is a central network zone that controls and inspects ingress and egress traffic between zones: the internet, on-premises, and spokes. The hub-and-spoke topology provides your IT department with an effective way to enforce security policies in a central location. It also reduces the potential for misconfiguration and exposure.

As shown in the diagram, Azure supports two types of hub-and-spoke designs:

- The first type supports communication, shared resources, and centralized security policies. This is labeled as the VNet hub in the diagram.
- The second type is based on Azure Virtual WAN and is labeled as Virtual WAN in the diagram. This type is designed for large-scale branch-to-branch and branch-to-Azure communications.


### be careful interconnecting between spokes
![image](/images/3.png)

A typical example of this scenario is the case where application processing servers are in one spoke or virtual network. The database deploys in a different spoke or virtual network. In this case, it's easy to interconnect the spokes with virtual network peering and avoid transiting through the hub. **Do a careful architecture and security review to ensure that bypassing the hub doesn't bypass important security or auditing points that are only in the hub.**

### The benefits of using a hub and spoke configuration include
![image](/images/4.png)


The benefits of using a hub and spoke configuration include:
- Cost savings
- Overcoming subscription limits
- Workload isolation
For more information, see [Hub-and-spoke network topology.](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/hub-spoke-network-topology)

When you peer virtual networks in different subscriptions, you can associate the subscriptions to the same or different Microsoft Entra tenants. This flexibility allows for decentralized management of each workload while maintaining shared services in the hub.

### Potential use cases
Typical uses for a hub and spoke architecture include workloads that:
- Have several environments that require shared services. For example, a workload might have development, testing, and production environments. Shared services might include DNS IDs, Network Time Protocol (NTP), or Active Directory Domain Services (AD DS). Shared services are placed in the hub virtual network, and each environment deploys to a different spoke to maintain isolation.
- Don't require connectivity to each other, but require access to shared services.
- Require central control over security, like a perimeter network (also known as DMZ) firewall in the hub with segregated workload management in each spoke.
- Require central control over connectivity, such as selective connectivity or isolation between spokes of certain environments or workloads.

### Recommendations
As a general rule, it's best to have at least one hub per region. This configuration helps avoid a single point of failure, for example to avoid Region A resources being affected at the network level by an outage in Region B.
#### GatewaySubnet
The virtual network gateway requires GatewaySubnet. You can also use a hub-spoke topology without a gateway if you don't need cross-premises network connectivity.
Create a subnet named GatewaySubnet with an address range of at least /27. The /27 address range gives the subnet enough scalability configuration options to prevent reaching the gateway size limitations in the future.
#### AzureFirewallSubnet
Create a subnet named AzureFirewallSubnet with an address range of at least /26. Regardless of scale, the /26 address range is the recommended size and covers any future size limitations. This subnet doesn't support network security groups (NSGs).

### Communication through an NVA
Communication via an NVA like a firewall and router. This method incurs a hop between the two spokes. If you need connectivity between spokes, consider deploying Azure Firewall or another NVA in the hub. Then create routes to forward traffic from a spoke to the firewall or NVA, which can then route to the second spoke. In this scenario, you must configure the peering connections to allow forwarded traffic.

![image](/images/5.png)

Evaluate the services you share in the hub to ensure that the hub scales for a larger number of spokes. For instance, if your hub provides firewall services, consider your firewall solution's bandwidth limits when you add multiple spokes. You can move some of these shared services to a second level of hubs.
The number of virtual network peerings per virtual network is limited. If you have many spokes that need to connect with each other, you could run out of peering connections. Connected groups also have limitations. In this scenario, consider using user-defined routes (UDRs) to force spoke traffic to be sent to Azure Firewall or another NVA that acts as a router at the hub. This change allows the spokes to connect to each other. To support this configuration, you must implement Azure Firewall with forced tunnel configuration enabled. The topology in this architectural design facilitates egress flows. While Azure Firewall is primarily for egress security, it can also be an ingress point,  but not recommended.

To configure spokes to communicate with remote networks through a hub gateway, you can use virtual network peerings or connected network groups.

To use virtual network peerings, in the virtual network Peering setup:

- Configure the peering connection in the hub to Allow gateway transit.
- Configure the peering connection in each spoke to Use the remote virtual network's gateway.
- Configure all peering connections to Allow forwarded traffic.

To use connected network groups:

1. In Virtual Network Manager, create a network group and add member virtual networks.
2. Create a hub and spoke connectivity configuration.
3. For the Spoke network groups, select Hub as gateway.

### Next steps
The hub in a hub-spoke network topology is the main component of a connectivity subscription in an [Azure landing zone](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/landing-zone). For more information about building large-scale networks in Azure with routing and security managed by the customer or by Microsoft, see [Define an Azure network topology](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/define-an-azure-network-topology).

## Enable vnet flow logging
Virtual network flow logging is a feature of Azure Network Watcher that allows you to log information about IP traffic flowing through an Azure virtual network.

### Register Insights provider
Microsoft.Insights provider must be registered to successfully log traffic flowing through a virtual network. If you aren't sure if the Microsoft.
Insights provider is registered, check its status in the Azure portal by following these steps:

1. In the search box at the top of the portal, enter subscriptions. Select Subscriptions from the search results.

![image](/images/6.png)

2. Select the Azure subscription that you want to enable the provider for in Subscriptions.

3. Under Settings, select Resource providers.

4. Enter insight in the filter box.

5. Confirm the status of the provider displayed is Registered. If the status is NotRegistered, select the Microsoft.Insights provider then select Register.

![image](/images/7.png)

### Create a flow log

Create a flow log for your virtual network, subnet, or network interface. This flow log is saved in an Azure storage account.

1. In the search box at the top of the portal, enter network watcher. Select Network Watcher from the search results.

2. Under Logs, select Flow logs.

3. In Network Watcher | Flow logs, select + Create or Create flow log blue button.

![image](/images/8.png)

4. On the Basics tab of Create a flow log, enter or select the following values:

| Setting          | Value                                                                                                                                                                                                                                                                                                                     | 
|------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Project details  |                                                                                                                                                                                                                                                                                                                           | 
| Subscription     | Select the Azure subscription of your virtual network that you want to log.                                                                                                                                                                                                                                               | 
| Flow log type    | Select Virtual network then select + Select target resource (available options are: Virtual network, Subnet, and Network interface).<br>Select the resources that you want to flow log, then select Confirm selection.                                                                                                    | 
| Flow Log Name    | Enter a name for the flow log or leave the default name. Azure portal uses {ResourceName}-{ResourceGroupName}-flowlog as a default name for the flow log.                                                                                                                                                                 | 
| Instance details |                                                                                                                                                                                                                                                                                                                           |
| Subscription     | Select the Azure subscription of the storage account.                                                                                                                                                                                                                                                                     | 
| Storage accounts | Select the storage account that you want to save the flow logs to. If you want to create a new storage account, select Create a new storage account.                                                                                                                                                                      | 
| Retention (days) | Enter a retention time for the logs (this option is only available with Standard general-purpose v2 storage accounts). Enter 0 if you want to retain the flow logs data in the storage account forever (until you manually delete it from the storage account). For information about pricing, see Azure Storage pricing. | 

![image](/images/9.png)

> If the storage account is in a different subscription, the resource that you're logging (virtual network, subnet, or network interface) and the storage account must be associated with the same Microsoft Entra tenant. The account you use for each subscription must have the necessary permissions.

5. To enable traffic analytics, select Next: Analytics button, or select the Analytics tab. Enter or select the following values:

| Setting                               | 	Value                                                                                                                                                                                                 |
|---------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Enable traffic analytics              | Select the checkbox to enable traffic analytics for your flow log.                                                                                                                                     |
| Traffic analytics processing interval | Select the processing interval that you prefer, available options are: Every 1 hour and Every 10 mins. The default processing interval is every one hour. For more information, see Traffic analytics. |
| Subscription                          | Select the Azure subscription of your Log Analytics workspace.                                                                                                                                         |
| Log Analytics Workspace               | Select your Log Analytics workspace. By default, Azure portal creates DefaultWorkspace-{SubscriptionID}-{Region} Log Analytics workspace in defaultresourcegroup-{Region} resource group.              |


![image](/images/10.png)

> To create and select a Log Analytics workspace other than the default one, see Create a Log Analytics workspace

6. Select Review + create.
7. Review the settings, and then select Create.















