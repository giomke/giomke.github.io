---
title: "Authentication in the Cloud"
date: 2025-04-01T08:12:04+00:00
tags: ["OIDC","FIDO2","WebAuthN","Identity","Federation","SAML2","WIF","passkey","passwordless","authentication"]
description: "Avoid using AWS IAM user access keys, Azure Service Principal client IDs/secrets, or Google Cloud Service Account JSON Web Tokens (JWTs)."
cover:
    image: "img/cover.png"
    alt: "Authentication in the Cloud"
    caption: "Avoid using AWS IAM user access keys, Azure Service Principal client IDs/secrets, or Google Cloud Service Account JSON Web Tokens (JWTs)."
    relative: true # when using page bundles set this to true
    hidden: true # only hide on current single page
---
# Introduction
I’ve noticed that many people in our field, even in 2025, still authenticate themselves and machines using old-fashioned way, such as passwords or long-term credentials. This short blog post takes elements from various resources to emphasize the importance of short-lived credentials for both humans and machines. The main focus of this blog is to promote the use of FIDO for user authentication and OIDC for machine authentication to avoid relying on long-term credentials.
# Background
As we all know, the oldest method we still use for authentication is the password. Over time, we've developed various techniques to protect passwords, such as encryption methods for securing data at rest and during transit. We've also started using passphrases and password managers, but the core issue of relying on passwords in the first place remains unchanged.

Instead of passwords, we can use certificates. Just like password-based technologies such as RADIUS, WPA2, NTLM, LDAP, HTTP Basic Authentication, and others, there are also many authentication methods based on certificates, such as SSH, VPN, Smart Cards, and many more. However, just like passwords, if a certificate is stolen, it can be used for unauthorized access.

After many breaches, phishing attacks, and leaked credentials, we began using two-factor authentication (2FA) methods such as TOTP, SMS, and email-based authentication. However, these methods are not phishing-resistant and can still be compromised using tools like Evilginx.

Following the rise of credential stuffing and credential reuse attacks, password managers and services like Have I Been Pwned gained popularity. People started using password managers, but in my observation, only a small portion of people are using them, to be honest. The problem of long-lived credentials and password sprawl, however, still persists.

The problem persists for two main reasons. First, not all services support 2FA, and second, many legacy protocols and machines can't use 2FA the way humans do. Despite these steps, attack vectors have dramatically decreased, but breaches still continue due to long-term, static credentials. Regardless of password managers, secrets are leaked in many cases and remain valid until rotated, which is often difficult and painful. Secrets can be leaked during debugging, in logs, shell history, repositories, artifact servers, third-party breaches, or by malware.

So, where do we stand today, and what are better ways to authenticate ourselves?

![image](/img/webauthn.png)


With the help of Trusted Platform Module (TPM), we can securely store our secrets and leverage FIDO2 and WebAuthn to use passkeys and passwordless authentication. These methods are phishing-resistant, more secure, and offer a much more comfortable experience in many aspects. Once we authenticate and identify ourselves, we can use our Identity Provider (IdP) as a single sign-on (SSO) solution through protocols like OIDC or SAML 2.0. This provides a high-level overview of modern authentication, in my view.

For personal use, for example, you can use Windows Hello to authenticate on Google (by registering a passkey) and then use Google as your Identity Provider for other sites—similar to the "Continue with Google" or "Sign in with your Google account" buttons on web login pages.

For businesses, you can use Windows Hello for Business to authenticate with your Azure Entra ID and use federation to access resources outside of your Azure environment.
# Authentication in the Cloud
## AWS
![image](/img/fido.png)

Both IAM Identity Center and AWS IAM support FIDO2 for authentication, providing a more secure and phishing-resistant method you can use.
**FIDO2** is a standard that includes **CTAP2** and **WebAuthn** and is based on public key cryptography. FIDO credentials are phishing-resistant because they are unique to the website that the credentials were created such as AWS.
AWS supports the two most common form factors for FIDO authenticators: built-in authenticators and security keys.
Many modern computers and mobile phones have **built-in authenticators**, such as TouchID on Macbook or a Windows Hello-compatible camera. 
If your device has a FIDO-compatible built-in authenticator, you can use your fingerprint, face, or device pin as a second factor. 
Security keys are FIDO-compatible **external hardware authenticators** that you can purchase and connect to your device through USB, BLE, or NFC. 
When you’re prompted for MFA, you simply complete a gesture with the key’s sensor. 
Some examples of security keys include YubiKeys and Feitian keys, and the most common security keys create device-bound FIDO credentials. 
For a list of all FIDO-certified security keys, see FIDO Certified Products.

For personal use, you can use AWS FIDO2 authentication, but for enterprise environments, the recommended approach is to rely on a centralized identity provider. AWS recommends **SEC02-BP04: Rely on a centralized identity provider**.

Since you might already have Azure Entra ID, it’s a good choice for AWS authentication. You can integrate an external identity source by connecting your identity provider via **SAML 2.0** and automatically provisioning users and groups using **SCIM**. Alternatively, you can connect to your Microsoft AD Directory using AWS Directory Service.

Once an identity source is configured, you can assign access to users and groups across AWS accounts by defining least-privilege policies in your permission sets. Your workforce users can then authenticate through your central identity provider to sign in to the AWS access portal and use **single sign-on (SSO)** for AWS accounts and cloud applications assigned to them.

Additionally, users can configure **AWS CLI v2** to authenticate with Identity Center and obtain credentials to run AWS CLI commands. Identity Center also enables **SSO access** to AWS applications such as Amazon SageMaker AI Studio and AWS IoT SiteWise Monitor portals.

## Azure
Imagine a world where users never type, change, or even know their passwords. Yes, it’s possible!
Users can sign in to Windows using Windows Hello for Business or FIDO2 security keys and seamlessly enjoy single sign-on (SSO) to Microsoft Entra ID and Active Directory resources.

Windows Hello is an authentication technology that allows users to sign in to their Windows devices using biometric data, or a PIN, instead of a traditional password. It provides enhanced security through phish-resistant two-factor authentication, and built-in brute force protection. With FIDO/WebAuthn, Windows Hello can also be used to sign in to supported websites, reducing the need to remember multiple complex passwords.

Likewise, we can authenticate to Microsoft Entra ID using a Microsoft Entra-joined device with Windows Hello for Business and use Microsoft Entra ID as a centralized identity provider to access resources outside our organization—for example, AWS resources.

![image](/img/windowsHellowFlow.png)

> All Microsoft Entra joined devices authenticate with Windows Hello for Business to Microsoft Entra ID the same way. The Windows Hello for Business trust type only impacts how the device authenticates to on-premises AD.

| Phase | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
|:-----:|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| A     | Authentication begins when the user dismisses the lock screen, which triggers Winlogon to show the Windows Hello for Business credential provider. The user provides their Windows Hello gesture (PIN or biometrics). The credential provider packages these credentials and returns them to Winlogon. Winlogon passes the collected credentials to LSASS. LSASS passes the collected credentials to the Cloud Authentication security support provider, referred to as the Cloud AP provider. |
| B     | The Cloud AP provider requests a nonce from Microsoft Entra ID. Microsoft Entra ID returns a nonce. The Cloud AP provider signs the nonce using the user's private key and returns the signed nonce to the Microsoft Entra ID.                                                                                                                                                                                                                                                                 |
| C     | Microsoft Entra ID validates the signed nonce using the user's securely registered public key against the nonce signature. Microsoft Entra ID then validates the returned signed nonce, and creates a PRT with session key that is encrypted to the device's transport key and returns it to the Cloud AP provider.                                                                                                                                                                            |
| D     | The Cloud AP provider receives the encrypted PRT with session key. Using the device's private transport key, the Cloud AP provider decrypt the session key and protects the session key using the device's TPM.                                                                                                                                                                                                                                                                                |
| E     | The Cloud AP provider returns a successful authentication response to LSASS. LSASS caches the PRT, and informs Winlogon of the success authentication. Winlogon creates a logon session, loads the user's profile, and starts explorer.exe.                                                                                                                                                                                                                                                    |

### Authenticate to AWS

To allow centralized identity management and avoid having to manage multiple identities and passwords, most organizations want to use single sign-on for platform resources. Some AWS customers rely on server-based Microsoft Active Directory for SSO integration. Other customers invest in third-party solutions to synchronize or federate their identities and provide SSO.

Microsoft Entra ID provides centralized identity management with strong SSO authentication. Almost any app or platform that follows common web authentication standards, including AWS, can use Microsoft Entra ID for identity and access management.

Many organizations already use Microsoft Entra ID to assign and protect Microsoft 365 or hybrid cloud identities. Employees use their Microsoft Entra identities to access email, files, instant messaging, cloud applications, and on-premises resources. You can quickly and easily integrate Microsoft Entra ID with your AWS accounts to let administrators and developers sign in to your AWS environments with their existing identities.

The following diagram shows how Microsoft Entra ID can integrate with multiple AWS accounts to provide centralized identity and access management:

![image](/img/federation.png)

Microsoft Entra ID offers several capabilities for direct integration with AWS:
- SSO across legacy, traditional, and modern authentication solutions.
- MFA, including integration with several third-party solutions from Microsoft Intelligent Security Association (MISA) partners.
- Powerful Conditional Access features for strong authentication and strict governance. Microsoft Entra ID uses Conditional Access policies and risk-based assessments to authenticate and authorize user access to the AWS Management Console and AWS resources.
- Large-scale threat detection and automated response. Microsoft Entra ID processes over 30 billion authentication requests per day, along with trillions of signals about threats worldwide.
- Privileged Access Management (PAM) to enable Just-In-Time (JIT) provisioning to specific resources.

We also have the capability for AWS Single-Account Access with Microsoft Entra ID. You can configure multiple identifiers for multiple instances.
AWS Single-Account Access has been used by customers over the past several years and enables you to federate Microsoft Entra ID to a single AWS account and use Microsoft Entra ID to manage access to AWS IAM roles. AWS IAM administrators define roles and policies in each AWS account. For each AWS account, Microsoft Entra administrators federate to AWS IAM, assign users or groups to the account, and configure Microsoft Entra ID to send assertions that authorize role access.

![image](/img/sso.png)

# Workload Identity Federation
We will discuss a great project by [Puma Security called Nymeria](https://pumasecurity.github.io/nymeria/), which is a workshop with ready-made files where you can learn and test machine-to-machine communication using Federation, all while avoiding long-term credentials.

Nymeria's goal is to help cloud identity and security teams to eliminate long-lived credentials from their cloud estate. The hands on workshop walks you through the following scenario:

1. A GitHub Action needs to authenticate to an Entra ID Tenant to run a Terraform deployment.

2. The Terraform deployment creates an Azure virtual machine that requires data stored in both AWS S3 and Google Cloud Storage (GCS).

![image](/img/oidc.png)

With the help of OIDC and SAML, we can authenticate workloads without the need for passwords or access keys. Workload Identity Federation is a cloud-native feature that allows authentication to public cloud APIs, helping us avoid the management of long-lived credentials and the associated complexities of securing them.

One important note to mention: Azure and GCP are OAuth-based and natively support OIDC, which means you can easily achieve workload identity federation. In contrast, AWS uses SigV2 and SigV4, with SigV4 support available in GCP but not in Azure. Azure does not support the AWS SigV4 exchange process like Google Cloud does. This means that you can authenticate AWS resources to access GCP without any issues, but you can't authenticate AWS resources to access Azure. On the other hand, AWS supports OIDC, allowing you to authenticate Azure, GCP, or other services like GitLab to access AWS resources, such as S3.

You can also create Kubernetes Workload Identity Federation for cross-cluster identity federation trust between Kubernetes service accounts running in AWS EKS, Azure AKS, and Google GKE. This federation trust allows a Kubernetes service account in any of the three clusters to access data stored across all three cloud storage services (S3, Azure Storage, and Google Cloud Storage).

![image](/img/oidc2.png)


# Conclusion
As you can see, we can easily achieve passwordless, seamless authentication for both humans and workloads. To do this, we need a centralized identity provider with strong authentication methods like FIDO for users and OIDC for workload authentication. In this setup, we no longer have to worry about credential rotations, password sprawl, or secret management. If you're not yet using FIDO2 and Workload Identity, now is the time to start.

# References

- [What is OAuth and OpenID Connect?](https://www.youtube.com/playlist?list=PLshTZo9V1-aEUg2S84KlisJBAyMEoEZ45)
- [Cross cloud workload identity research and workshops](https://github.com/pumasecurity/nymeria/tree/main)
- [SEC02-BP04 Rely on a centralized identity provider](https://docs.aws.amazon.com/wellarchitected/2022-03-31/framework/sec_identities_identity_provider.html#:~:text=For%20workforce%20identities%2C%20rely%20on%20an%20identity%20provider,managing%2C%20and%20revoking%20access%20from%20a%20single%20location.)
- [Supported configurations for using passkeys and security keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_fido_supported_configurations.html)
- [Available MFA types for IAM Identity Center](https://docs.aws.amazon.com/singlesignon/latest/userguide/mfa-types.html)
- [Microsoft Entra identity management and access management for AWS](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/aws/aws-azure-ad-security)
- [Microsoft Entra SSO integration with AWS Single-Account Access](https://learn.microsoft.com/en-us/entra/identity/saas-apps/amazon-web-service-tutorial)
- [Windows Hello for Business authentication](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/how-it-works-authentication)


