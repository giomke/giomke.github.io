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
















