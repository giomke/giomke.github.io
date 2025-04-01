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
