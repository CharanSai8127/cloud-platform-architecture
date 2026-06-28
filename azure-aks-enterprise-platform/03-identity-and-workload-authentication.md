# Identity And Workload Authentication

## Overview

Identity has become the foundation of modern cloud security. Traditional application architectures relied heavily on usernames, passwords, API keys, certificates, and long-lived connection strings to authenticate with external services. While functional, this approach introduces significant operational and security challenges, including credential leakage, difficult secret rotation, excessive privilege, and manual lifecycle management.

Modern cloud platforms adopt an identity-first security model where workloads authenticate themselves rather than presenting static credentials. Instead of distributing secrets to every application, trusted identities are established and verified by a centralized identity provider before access to cloud resources is granted.

This project adopts Microsoft Entra ID together with Azure Workload Identity to establish secure, identity-based authentication between Azure Kubernetes Service (AKS) workloads and Azure-managed services such as Azure SQL Database and Azure Key Vault.

---

# Why Identity Has Become The New Security Perimeter

Applications rarely operate independently. They continuously communicate with external services including databases, secret management systems, storage platforms, messaging services, and monitoring platforms.

Historically, applications authenticated using:

- Usernames and Passwords
- API Keys
- Certificates
- Connection Strings
- Static Access Tokens

Although these methods enable communication, they introduce several challenges.

Examples include:

- Long-lived Credentials
- Secret Leakage
- Difficult Credential Rotation
- Hardcoded Configuration
- Excessive Privileges
- Increased Operational Overhead

As enterprise systems continue to grow, managing thousands of credentials across multiple applications becomes increasingly complex.

Modern cloud platforms therefore shift the focus from authenticating credentials to authenticating identities.

Identity becomes the new security perimeter.

---

# Identity Challenges In Kubernetes

Applications deployed within Kubernetes frequently require access to Azure-managed services.

Examples include:

- Azure Key Vault
- Azure SQL Database
- Azure Storage
- Azure Monitor
- Azure APIs

A fundamental challenge arises:

**How does Azure determine which Kubernetes workload is requesting access?**

Unlike traditional virtual machines, Kubernetes Pods are dynamic.

Pods are continuously:

- Created
- Deleted
- Rescheduled
- Replaced

Assigning long-lived credentials to every Pod would introduce unnecessary security risks.

Instead, workloads should authenticate using their identity rather than possessing static credentials.

---

# Microsoft Entra ID

Microsoft Entra ID serves as the centralized identity provider for Azure resources.

Every managed Azure service relies on Microsoft Entra ID to perform:

- Authentication
- Authorization
- Identity Management
- Auditing
- Governance

Resources such as Azure SQL Database and Azure Key Vault are tightly integrated with Microsoft Entra ID, ensuring that every request originates from an authenticated and authorized identity.

Rather than trusting applications, Azure trusts identities that have been explicitly granted permissions.

---

# Deployment Identity Using Service Principal

Infrastructure provisioning also requires authentication.

Terraform cannot create Azure resources anonymously.

Instead, Terraform authenticates with Azure Resource Manager using a Service Principal.

The Service Principal acts as the deployment identity responsible for provisioning and managing cloud infrastructure.

```text
Terraform
        │
        ▼
Service Principal
        │
        ▼
Azure Resource Manager
        │
        ▼
Azure Resources
```

Using a Service Principal provides several advantages:

- Authenticated Infrastructure Provisioning
- Controlled Permissions
- Auditability
- Governance
- Resource Lifecycle Management

The Service Principal is responsible only for deploying infrastructure.

It is **not** used by the application at runtime.

---

# Runtime Identity Using User Assigned Managed Identity

Once the infrastructure has been provisioned, application workloads require their own identity.

Rather than distributing credentials to every workload, Azure provides Managed Identities.

This project adopts a **User Assigned Managed Identity (UAMI)**.

A User Assigned Managed Identity is an Azure-managed identity that exists independently of any individual resource and can be associated with one or more Azure services.

The managed identity represents the workload's identity within Microsoft Entra ID.

Instead of storing passwords or client secrets, the managed identity authenticates directly with Azure services.

Its responsibilities include:

- Identity Representation
- Azure Authentication
- Azure RBAC Authorization
- Secure Service Access

The managed identity answers a simple question:

**Who is requesting access?**

It does not determine **whether** access should be granted.

Authorization remains the responsibility of Azure Role-Based Access Control and the target Azure service.

---

# OpenID Connect (OIDC)

Kubernetes and Microsoft Entra ID operate as independent identity systems.

Kubernetes authenticates workloads using Service Accounts.

Azure authenticates workloads using Microsoft Entra ID identities.

A trust mechanism is therefore required to bridge these two identity systems.

Azure Workload Identity relies on **OpenID Connect (OIDC)** to establish this trust.

The AKS cluster exposes an OIDC issuer capable of issuing signed identity tokens for Kubernetes Service Accounts.

These tokens become verifiable proof of workload identity.

---

# Federated Identity Credential

Possessing an OIDC token alone is insufficient.

Azure must still determine whether the requesting Kubernetes workload is permitted to assume a Managed Identity.

This trust relationship is established through a **Federated Identity Credential**.

The Federated Identity Credential maps:

- Kubernetes Service Account
- Kubernetes Namespace
- OIDC Issuer
- User Assigned Managed Identity

Only workloads matching this configuration are allowed to exchange their Kubernetes-issued identity token for an Azure access token.

Conceptually, the Federated Identity Credential performs a role similar to the trust policy used by AWS IAM Roles for Service Accounts (IRSA).

Its responsibility is to answer:

**Which Kubernetes workload is allowed to assume this Managed Identity?**

---

# Kubernetes Service Account

Within Kubernetes, every workload executes using a Service Account.

The Service Account becomes the workload's identity inside the cluster.

To participate in Azure Workload Identity, the Service Account is annotated with the Client ID of the User Assigned Managed Identity.

The annotation informs Azure Workload Identity which Managed Identity should be requested on behalf of the workload.

The Service Account itself does not grant Azure permissions.

Instead, it identifies the workload that intends to authenticate with Azure.

---

# Azure Workload Identity Authentication Flow

The complete authentication sequence is illustrated below.

```text
Application Pod
        │
        ▼
Kubernetes Service Account
        │
        ▼
OIDC Token Issued By AKS
        │
        ▼
Federated Identity Credential
        │
        ▼
User Assigned Managed Identity
        │
        ▼
Microsoft Entra ID
        │
        ▼
Azure Access Token
        │
        ▼
Azure Key Vault
```

No passwords, client secrets, or long-lived credentials are exchanged during this process.

Authentication is entirely identity-driven.

---

# Identity Architecture Across The Platform

The platform separates identities according to their operational responsibilities.

## Service Principal

Responsible for:

- Infrastructure Deployment
- Resource Provisioning
- Azure Resource Manager Authentication

---

## User Assigned Managed Identity

Responsible for:

- Runtime Identity
- Azure Authentication
- Secure Resource Access

---

## Federated Identity Credential

Responsible for:

- Trust Relationship
- OIDC Federation
- Identity Mapping

---

## Kubernetes Service Account

Responsible for:

- Workload Identity Within Kubernetes
- Managed Identity Association

---

## Microsoft Entra ID

Responsible for:

- Identity Verification
- Authentication
- Authorization
- Token Issuance

Each identity component performs a single responsibility, reducing complexity while improving security.

---

# Security Benefits

Adopting Azure Workload Identity provides several advantages over traditional credential-based authentication.

These include:

- Elimination Of Static Credentials
- Reduced Secret Distribution
- Centralized Identity Management
- Fine-Grained Access Control
- Short-Lived Authentication Tokens
- Improved Auditability
- Least Privilege Access
- Simplified Credential Rotation

Rather than embedding credentials within application manifests, workloads prove their identity before Azure grants access.

---

# Trade-Offs

Identity-based authentication significantly improves platform security but introduces additional architectural complexity.

Examples include:

- OIDC Configuration
- Managed Identity Management
- Federated Identity Configuration
- Azure RBAC Planning
- Additional Platform Resources

The project intentionally accepts this complexity because identity-based authentication is considerably more secure than distributing long-lived credentials across application workloads.

---

# Summary

Modern enterprise platforms increasingly rely on identity rather than credentials to secure communication between workloads and managed cloud services.

This project adopts Microsoft Entra ID, Azure Workload Identity, User Assigned Managed Identities, Federated Identity Credentials, and Kubernetes Service Accounts to establish secure, identity-based authentication throughout the platform.

The Service Principal provisions infrastructure, the Managed Identity represents the workload at runtime, the Federated Identity Credential establishes trust between Kubernetes and Azure, and Microsoft Entra ID verifies every authentication request before access to Azure resources is granted.

This identity foundation enables the platform to securely consume Azure SQL Database, Azure Key Vault, and other Azure-native services without distributing static credentials, establishing a modern, scalable, and enterprise-ready security model.
