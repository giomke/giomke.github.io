---
title: "AWS security baseline architecture"
date: 2024-07-02T11:30:03+00:00
tags: ["AWS","architecture", "SRA"]
description: "giomke personal blog"

---
# AWS security baseline architecture
![image](/images/1.png)


## Introduction
Hi there in this blog post, we will talk about AWS cloud security architecture from a thousand-foot View. This blog post is meant for absolute beginners in the cloud security journey. We will discuss how we can achieve perfect isolation of resources and collaboration with each other by following best practices and recommended guides.

AWS offers numerous security-related resources, including posts, whitepapers, and guides, as well as two major frameworks: the Cloud Adoption Framework (CAF) and the Well-Architected Framework. 
However, the abundance of information can make AWS security seem complex and confusing for beginners like myself, and possibly like you. It's likely that AWS developed the AWS Security Reference Architecture (AWS SRA) in response to this confusion. As its name suggests, the SRA organizes best practices, whitepapers, and guidelines into a cohesive framework, making them easier to follow and implement. According to Amazon, the SRA provides a holistic set of guidelines for deploying AWS security services in a multi-account environment. Before diving into the details of the SRA, let's first understand why a multi-account strategy is necessary and why prioritizing security services matters.

![image](/images/2.png)


## Multi-account strategy the best way to achieve isolation
Frankly speaking, you can achieve your goals with one AWS account but the question arises why do we need multiple accounts, and why does even AWS recommend us to follow a multi-account strategy? It has a couple of answers but let's talk from a security perspective.
As security professionals, we prioritize isolating resources for security reasons. It's likely you've already grasped the concept that the most effective way to isolate AWS resources and minimize the 'blast radius' is by leveraging multiple accounts. 

![recommended-ous](/images/3.png)


For example, let's consider a scenario where we have two accounts: one for development resources and another for production resources, instead of having just one account. In the event of an incident, suppose an EC2 machine is compromised in the DEV account due to weaker security measures compared to the PROD account. In the worst-case scenario, the attacker abuse the AWS metadata service located at 169.254.169.254. Leveraging the permissions set to the compromised EC2 instance, the attacker manages to create a new user in AWS IAM and assigns the admin role to this user. They then log out from the compromised EC2 instance and log in to the AWS console as an admin with full permissions.

![metadata server attack](/images/4.gif)


However, with a multi-account strategy in place, the attacker still wouldn't have access to the PROD account, where our most critical assets are stored. This underscores the importance of account-level isolation because even in the event of a full compromise, the adversary would still not have access to other accounts, theoretically. It's important to note that the effectiveness of this strategy depends on how the environment is configured, but as of now, it stands as the optimal choice for isolating resources. To simplify, remember that each AWS account acts as a distinct repository for AWS resources and serves no other purpose.

Organizing multiple accounts and managing them efficiently was indeed a challenging task until the introduction of the AWS Organizations service. This service, which is free to use, has streamlined the process of managing multiple AWS accounts effectively. In the next section, we'll delve deeper into the features and benefits of AWS Organizations and how it simplifies the management of multi-account architectures.

![image](/images/5.png)


## Grouping and managing multi-account organization
IT professionals like to group assets because it makes management and governance easier. That's why AWS recommends having users in user groups and attaching security policies to a group instead of directly to users. It is similar in accounts; we need to have the opportunity to group them logically and apply policies. To achieve our goals, AWS offers a service named AWS Organizations, which we can use, and it's free, by the way. AWS Organizations is an account management service that enables you to consolidate multiple AWS accounts into an organization that you create and centrally manage. AWS Organizations includes account management and consolidated billing capabilities that enable you to better meet the budgetary, security, and compliance needs of your business. As an administrator of an organization, you can create accounts in your organization and invite existing accounts to join the organization.

![image](/images/6.png)

Regardless of numerous recommendations on how to organize your accounts and structure your organization, the question still arises in some situations. In the following sections, I will provide you with a blueprint from the SRA, but remember, the core mindset is resource-based grouping. For example, as mentioned before, the account is a box of resources, and with the help of AWS Organizations OUs, we have to group the same resource boxes, in this case, the same resource-based accounts. You will achieve a cleaner architecture when you group based on the same resource accounts instead of grouping them based on users, teams, or your company structure. When you group based on the same resource account, you can deploy more robust and effective Service Control Policies (SCPs). Service Control Policies (SCPs) are a type of organizational policy that you can use to manage permissions in your organization. SCPs offer central control over the maximum available permissions for the IAM users and IAM roles in your organization. SCPs help you ensure your accounts stay within your organization’s access control guidelines.

Managing the AWS Organization service can be tedious because it requires manual effort in many areas. For this reason, AWS offers a service named AWS Control Tower, which utilizes services like AWS Organizations, AWS IAM Identity Center, AWS Config, and perhaps more that I don't remember right now. AWS Control Tower provides a straightforward way to set up and govern an AWS multi-account environment, following prescriptive best practices. Resources are set up and managed on your behalf. I recommend using this service at the beginning of your cloud journey because it establishes a basic security landing zone where you can align with the following guidelines of security architecture.

![image](/images/7.png)

## Organization Units

AWS Organizations is a service that helps you centrally manage and govern your AWS environment. It enables you to centrally manage and govern multiple AWS accounts. Within AWS Organizations, you can organize these accounts into organizational units (OUs).
By structuring your AWS accounts into organizational units, you can more effectively manage your resources, control access to services and resources, and apply policies consistently across multiple accounts. This helps improve security, compliance, and cost management within your AWS environment.

![image](/images/8.png)

### First four account for your secure landing zone
A Secure Landing Zone (SLZ) is a foundational environment in AWS that is designed with security best practices in mind. It serves as a secure and compliant starting point for hosting applications and workloads in the cloud.
My recomendation is to have at least four account in the begining: Org Management account, Security Tooling account, Network account and Application account.
> [Descriptions for the following accounts come from The AWS Security Reference Architecture. For more in-depth information, please visit The AWS Security Reference Architecture documentation.](https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/architecture.html)

#### The management account, trusted access, and delegated administrators
The management account (also called the AWS Organization Management account or Org Management account) is unique and differentiated from every other account in AWS Organizations. It is the account that creates the AWS organization. From this account, you can create AWS accounts in the AWS organization, invite other existing accounts to the AWS organization (both types are considered member accounts), remove accounts from the AWS organization, and apply IAM policies to the root, OUs, or accounts within the AWS organization. 

The management account deploys universal security guardrails through SCPs and service deployments (such as AWS CloudTrail) that will affect all member accounts in the AWS organization. To further restrict permissions in the management account, those permissions can be delegated to another appropriate account, such as a security account, where possible. 

The management account has the responsibilities of a payer account and is responsible for paying all charges that are accrued by the member accounts. You cannot switch an AWS organization's management account. An AWS account can be a member of only one AWS organization at a time.   

Because of the functionality and scope of influence the management account holds, we recommend that you limit access to this account and grant permissions only to roles that need them. Two features that help you do this are trusted access and delegated administrator. You can use trusted access to enable an AWS service that you specify, called the trusted service, to perform tasks in your AWS organization and its accounts on your behalf. This involves granting permissions to the trusted service but does not otherwise affect the permissions for IAM entities. You can use trusted access to specify settings and configuration details that you would like the trusted service to maintain in your AWS organization's accounts on your behalf. 

Some AWS services support the delegated administrator feature in AWS Organizations. With this feature, compatible services can register an AWS member account in the AWS organization as an administrator for the AWS organization's accounts in that service. This capability provides flexibility for different teams within your enterprise to use separate accounts, as appropriate for their responsibilities, to manage AWS services across the environment. The AWS security services in the AWS SRA that currently support delegated administrator include AWS IAM Identity Center (successor to AWS Single Sign-On), AWS Config, AWS Firewall Manager, Amazon GuardDuty, AWS IAM Access Analyzer, Amazon Macie, AWS Security Hub, Amazon Detective, AWS Audit Manager, Amazon Inspector, and AWS Systems Manager. Use of the delegated administrator feature is emphasized in the AWS SRA as a best practice, and we delegate administration of security-related services to the Security Tooling account.

![image](/images/9.png)

#### Security Tooling account
In the context of AWS services, this account is used to provide centralized delegated admin access to AWS security tooling and consoles, as well as provide view-only access for investigative purposes into all accounts in the organization. The security tooling account should be restricted to authorized security and compliance personnel and related security. This account is an aggregation point (or points for organizations that split the functionality across multiple accounts) for AWS security services, including AWS Security Hub, Amazon GuardDuty, Amazon Macie, AWS AppConfig, AWS Firewall Manager, Amazon Detective, Amazon Inspector, and IAM Access Analyzer.

The Security Tooling account is dedicated to operating security services, monitoring AWS accounts, and automating security alerting and response. The security objectives include the following:

- Provide a dedicated account with controlled access to manage access to the security guardrails, monitoring, and response.
- Maintain the appropriate centralized security infrastructure to monitor security operations data and maintain traceability. Detection, investigation, and response are essential parts of the security lifecycle and can be used to support a quality process, a legal or compliance obligation, and for threat identification and response efforts.
- Further support a defense-in-depth organization strategy by maintaining another layer of control over appropriate security configuration and operations such as encryption keys and security group settings. This is an account where security operators work. Read-only/audit roles to view AWS organization-wide information are typical, whereas write/modify roles are limited in number, tightly controlled, monitored, and logged.

![image](/images/10.png)

#### Network account
The Network account serves as the central hub for your network within your AWS Organization. You can manage your networking resources and route traffic between accounts in your environment, your on-premises, and egress/ingress traffic to the internet. Within this account, your network administrators can manage and build security measures to protect network traffic across your cloud environment.

The Network account manages the gateway between your application and the broader internet. It is important to protect that two-way interface. The Network account isolates the networking services, configuration, and operation from the individual application workloads, security, and other infrastructure. This arrangement not only limits connectivity, permissions, and data flow, but also supports separation of duties and least privilege for the teams that need to operate in these accounts. By splitting network flow into separate inbound and outbound virtual private clouds (VPCs), you can protect sensitive infrastructure and traffic from undesired access. The inbound network is generally considered higher risk and deserves appropriate routing, monitoring, and potential issue mitigations. These infrastructure accounts will inherit permission guardrails from the Org Management account and the Infrastructure OU. Networking (and security) teams manage the majority of the infrastructure in this account.

![image](/images/11.png)

#### Application account
The Application account hosts the primary infrastructure and services to run and maintain an enterprise application. The Application account and Workloads OU serve a few primary security objectives. First, you create a separate account for each application to provide boundaries and controls between workloads so that you can avoid issues of comingling roles, permissions, data, and encryption keys. You want to provide a separate account container where the application team can be given broad rights to manage their own infrastructure without affecting others. Next, you add a layer of protection by providing a mechanism for the security operations team to monitor and collect security data. Employ an organization trail and local deployments of account security services (Amazon GuardDuty, AWS Config, AWS Security Hub, Amazon EventBridge, AWS IAM Access Analyzer), which are configured and monitored by the security team. Finally, you enable your enterprise to set controls centrally. You align the application account to the broader security structure by making it a member of the Workloads OU through which it inherits appropriate service permissions, constraints, and guardrails.

![image](/images/12.png)

### Security OU, Infrastructure OU, Workloads OU
In this chapter, let's group the previously described accounts. I will show which organization unit each account belongs to (management account does not need OU).

#### Security OU
The Security OU is a foundational OU. Your security organization should own and manage this OU along with any child OUs and associated accounts. You create Security Tooling account in the Security OU. A default deployment of AWS Control Tower will create a Log Archive and Audit (also referred to as Security Tooling) accounts.
#### Infrastructure OU
The Infrastructure OU is a foundational OU that is intended to contain infrastructure services. The accounts in this OU are also considered administrative and your infrastructure and operations teams should own and manage this OU, any child OUs, and associated accounts. The Infrastructure OU is used to hold AWS accounts containing AWS infrastructure resources that are shared, utilized by, or used to manage accounts in the organization. This includes centralized operations or monitoring of your organization. No application accounts or application workloads are intended to exist within this OU. Common use cases for this OU include accounts to centralize management of resources. For example, a Network account might be used to centralize your AWS network, or an Operations Tooling account to centralize your operational tooling. You create Network account in the Infrastructure OU.

In most cases, given the way most AWS Organization integrated services interact with the accounts within the Infrastructure OU, it does not generally make sense to have production and non-production variants of these accounts within the Infrastructure OU. In situations where non-production accounts are required, these workloads should be treated like any other application and placed in an account within the appropriate Workloads OU corresponding with the non-production phase of the SDLC (Dev OU or Test OU).

#### Workloads OU
The Workloads OU is intended to house most of your business-specific workloads including both production and non-production environments. These workloads can be a mix of commercial off-the-shelf (COTS) applications and your own internally developed custom applications and data services. Workloads in this OU often include shared application and data services that are used by other workloads. You create Application account in the Workloads OU.

![image](/images/13.png)

## Conclusion
In conclusion, understanding AWS cloud security architecture is crucial for ensuring the protection of resources and data in the cloud environment. This blog post has provided a beginner-friendly overview, introducing key concepts such as the AWS Security Reference Architecture (AWS SRA), multi-account strategy, and the role of AWS Organizations in managing accounts efficiently.

By emphasizing the importance of resource isolation and collaboration through best practices, the post highlights the significance of structuring AWS accounts logically and deploying robust security measures. The recommended Secure Landing Zone (SLZ) framework, consisting of Org Management, Security Tooling, Network, and Application accounts, serves as a foundational setup for building a secure AWS environment.

Furthermore, the concept of organizational units (OUs) within AWS Organizations offers a structured approach to account management, allowing for consistent application of security policies across multiple accounts.

Overall, this post provides a foundational understanding of AWS cloud security architecture, laying the groundwork for beginners to navigate and implement security best practices effectively in their AWS environments. With a solid understanding of these principles, users can embark on their cloud security journey with confidence, knowing they have the tools and knowledge to protect their assets in the cloud effectively.

## References
- [AWS Security Reference Architecture](https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/welcome.html)
- [Organizing Your AWS Environment Using Multiple Accounts](https://docs.aws.amazon.com/whitepapers/latest/organizing-your-aws-environment/organizing-your-aws-environment.html)
- [AWS Security Reference Architecture: Visualize your security - Mohamed Wali - NDC Security 2024](https://www.youtube.com/watch?v=jGxS-7s-0sE)
- [AWS re:Inforce 2023 - Plan and deploy your own security architecture based on the AWS SRA (GRC307)](https://www.youtube.com/watch?v=ayvGswS72x4)
- [AWS re:Invent 2022 - Revitalize your security with the AWS Security Reference Architecture (SEC203)](https://www.youtube.com/watch?v=uFrj0jHN848)
- [AWS re:Invent 2021 - AWS Security Reference Architecture: Visualize your security](https://www.youtube.com/watch?v=-_LzB1uibAs)
- [AWS re:Invent 2018: Architecting Security & Governance across your AWS Landing Zone (SEC303-R1)](https://www.youtube.com/watch?v=Fxkbz0OwPKk)



