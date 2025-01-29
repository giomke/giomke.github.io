---
title: "Azure Network Security Monitoring"
date: 2024-1-29T20:00:04+00:00
tags: ["Azure","KQL","Detection","Scan","log","flow","hunting","network","vnet","alert"]
description: "detecting horizontal and vertical scans"
---
## Introduction:
In today’s dynamic cloud environments, detecting and mitigating network scans is crucial for ensuring the security and health of your infrastructure. Azure provides robust tools like VNet Flow Logs, Log Analytics, and custom KQL (Kusto Query Language) rules that can help you detect and respond to both horizontal and vertical network scans. In this post, we will dive into how you can leverage these tools to build effective detection mechanisms for network scans in Azure. We will cover key concepts like VNet flow logs, network architecture best practices, and provide step-by-step guidance on creating custom KQL queries to identify scanning activities in your environment. Whether you’re an Azure security enthusiast or a network administrator, this guide will empower you to enhance your cloud network monitoring.

##  VNet flow logs
