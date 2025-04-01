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
