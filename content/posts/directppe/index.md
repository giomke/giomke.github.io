---
title: "Credential Theft"
date: 2025-09-27T08:15:04+00:00
---
# Credential Theft via Direct Poisoned Pipeline Execution (Direct-PPE) Attacks on AWS, Azure, and GCP
In this short blog post, I’ll show how an attacker with existing access to a source code management (SCM) system can abuse malicious pipeline configurations to steal cloud tokens and escalate access across AWS, Azure, or GCP environments.

## Direct-PPE
First, let’s clarify what Direct Poisoned Pipeline Execution (Direct-PPE) means and outline the breach scenario we’ll use to demonstrate how attackers can leverage it to further compromise cloud environments like AWS, Azure, and GCP.

Poisoned Pipeline Execution (PPE) risks refer to the ability of an attacker with access to source control systems - and without access to the build environment, to manipulate the build process by injecting malicious code/commands into the build pipeline configuration, essentially ‘poisoning’ the pipeline and running malicious code as part of the build process.

We have three types of PPE
- Direct PPE (D-PPE) : This occurs when an attacker gains direct access to the pipeline configuration. For example, they might obtain a personal access token (PAT), valid user credentials, or even a session cookie for the SCM platform, giving them the ability to modify and execute malicious changes in the pipeline configuration.
- Indirect PPE (I-PPE): This means the attacker doesn’t have direct access to the pipeline configuration files but can still compromise the pipeline by injecting malicious code into files referenced by the configuration. Examples include Makefiles, test code, scripts, or automation tools that the pipeline relies on during execution.
- Public-PPE (3PE): This is often the most dangerous scenario. If the CI pipeline of a public repository automatically runs unreviewed code submitted by anonymous contributors, it becomes vulnerable to a Public PPE (3PE) attack. This can lead to exposure of internal assets—such as secrets from private projects—especially when the public repository’s pipeline shares the same CI instance or infrastructure as private repositories.

In the next section, we’ll assume the breach has already occurred, the attacker has access to the SCM, access by having an authentication method to the organization’s source code management. It may be a personal access token (PAT), an SSH key, or any other allowed authentication credential. An example of a method an attacker can use to achieve this technique is using a phishing attack against an organization. We’ll explore how they can escalate privileges further to gain access to cloud environments.
