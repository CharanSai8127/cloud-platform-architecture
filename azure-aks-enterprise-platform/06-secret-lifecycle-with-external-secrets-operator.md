# Secret Lifecycle With External Secrets Operator

## Overview

Centralizing secrets in Azure Key Vault solves only part of the secret management problem. Applications deployed within Kubernetes cannot directly consume secrets stored inside Azure Key Vault because Kubernetes workloads expect secrets to exist as native Kubernetes Secret resources.

Simply allowing every application to communicate directly with Azure Key Vault would tightly couple application logic with the cloud provider, increase authentication complexity, and require every workload to independently manage secret retrieval.

Instead, this project adopts the External Secrets Operator (ESO) as the bridge between Azure Key Vault and Kubernetes.

The External Secrets Operator continuously synchronizes secrets from Azure Key Vault into Kubernetes Secrets while preserving centralized governance, identity-based authentication, and Kubernetes-native application consumption.

---

# The Secret Consumption Problem

Applications require configuration such as:

- Database Connection Strings
- API Keys
- Authentication Tokens
- Certificates
- Encryption Keys

These secrets are securely stored inside Azure Key Vault.

However, Kubernetes applications typically consume configuration through:

- Environment Variables
- Kubernetes Secrets
- Mounted Secret Volumes

This creates an architectural gap.

Azure Key Vault manages secrets.

Kubernetes applications consume Kubernetes Secrets.

A synchronization mechanism is therefore required.

---

# Why Not Allow Applications To Read Azure Key Vault Directly?

Although technically possible, allowing every application to directly communicate with Azure Key Vault introduces several challenges.

Every application would become responsible for:

- Azure Authentication
- Secret Retrieval
- Secret Refresh
- Retry Logic
- Failure Handling
- Secret Version Management

This tightly couples application logic with cloud-specific APIs and increases operational complexity.

Instead, secret retrieval should become a platform responsibility rather than an application responsibility.

Applications should simply consume configuration without knowing where the secret originated.

---

# External Secrets Operator

The External Secrets Operator (ESO) is a Kubernetes operator responsible for synchronizing secrets from external secret management systems into Kubernetes.

Rather than applications communicating directly with Azure Key Vault, ESO performs the communication on behalf of the cluster.

The operator continuously reconciles the desired secret configuration and ensures Kubernetes Secrets remain synchronized with Azure Key Vault.

This follows Kubernetes' reconciliation model, where the system continuously works towards the desired state instead of relying on one-time synchronization.

---

# Why An Operator?

Kubernetes is built around reconciliation.

Native Kubernetes controllers continuously monitor cluster resources and reconcile the current state with the desired state stored in the Kubernetes API.

The External Secrets Operator extends this reconciliation model to external secret providers.

Instead of managing Deployments or StatefulSets, it manages secret synchronization.

Whenever:

- A secret changes in Azure Key Vault
- A Kubernetes Secret is deleted
- A new ExternalSecret is created

The operator automatically reconciles the desired state.

Applications therefore remain synchronized without manual intervention.

---

# ClusterSecretStore

The External Secrets Operator requires knowledge of how to communicate with Azure Key Vault.

This responsibility belongs to the ClusterSecretStore resource.

The ClusterSecretStore defines:

- Secret Provider
- Authentication Method
- Azure Key Vault Instance
- Identity Configuration

Rather than configuring every application independently, the platform defines a centralized secret provider that can be shared across multiple namespaces and workloads.

This improves consistency while reducing duplication.

---

# ExternalSecret

While the ClusterSecretStore defines **where** secrets originate, the ExternalSecret resource defines **which** secrets should be synchronized.

Each ExternalSecret specifies:

- Azure Key Vault Secret
- Kubernetes Secret Name
- Synchronization Interval
- Target Namespace

The External Secrets Operator continuously watches these resources and ensures the corresponding Kubernetes Secret always matches the latest version stored within Azure Key Vault.

---

# Kubernetes Secret

Applications are intentionally isolated from Azure Key Vault.

Instead of communicating directly with Azure services, applications consume standard Kubernetes Secrets.

From the application's perspective, nothing changes.

The application simply references:

- Environment Variables
- Secret References
- Mounted Secret Volumes

The application remains completely unaware that the secret originated from Azure Key Vault.

This separation significantly simplifies application development while preserving centralized secret governance.

---

# Identity-Based Secret Synchronization

The External Secrets Operator itself does not store Azure credentials.

Instead, it authenticates using Azure Workload Identity.

The authentication flow follows the identity architecture established earlier.

The operator:

- Uses a Kubernetes Service Account
- Receives an OIDC token
- Exchanges the token through the Federated Identity Credential
- Assumes the User Assigned Managed Identity
- Authenticates with Microsoft Entra ID
- Retrieves secrets from Azure Key Vault

No static credentials are stored within Kubernetes.

Every authentication request is identity-driven.

---

# Complete Secret Lifecycle

The complete lifecycle becomes:

```text
Terraform
        │
        ▼
Azure Key Vault
        │
        ▼
Secret Created
        │
        ▼
Microsoft Entra ID
        │
        ▼
User Assigned Managed Identity
        │
        ▼
Federated Identity Credential
        │
        ▼
External Secrets Operator
        │
        ▼
ClusterSecretStore
        │
        ▼
ExternalSecret
        │
        ▼
Kubernetes Secret
        │
        ▼
.NET Application
```

Every component performs a single responsibility while collectively establishing a secure and automated secret management workflow.

---

# Separation Of Responsibilities

One of the primary design principles of this architecture is separating platform responsibilities from application responsibilities.

## Azure Key Vault

Responsible for:

- Secret Storage
- Secret Rotation
- Secret Versioning
- Governance
- Auditing

---

## Microsoft Entra ID

Responsible for:

- Authentication
- Identity Verification
- Token Issuance

---

## Azure Workload Identity

Responsible for:

- Secure Workload Authentication
- Identity Federation

---

## External Secrets Operator

Responsible for:

- Secret Synchronization
- Continuous Reconciliation
- Kubernetes Secret Management

---

## Kubernetes

Responsible for:

- Secret Distribution
- Application Configuration

---

## Application

Responsible only for:

- Consuming Configuration
- Executing Business Logic

The application has no knowledge of Azure Key Vault, Microsoft Entra ID, or secret synchronization.

This separation significantly reduces application complexity while strengthening platform security.

---

# Continuous Reconciliation

Unlike one-time synchronization mechanisms, the External Secrets Operator continuously monitors the desired state.

Whenever:

- A secret changes
- A secret is rotated
- A Kubernetes Secret is deleted
- An ExternalSecret is updated

The operator automatically reconciles the Kubernetes Secret.

This ensures the cluster remains synchronized with Azure Key Vault without requiring manual intervention or application redeployment.

---

# Advantages

The External Secrets architecture provides several operational advantages.

These include:

- Centralized Secret Management
- Identity-Based Authentication
- Continuous Reconciliation
- Kubernetes-Native Secret Consumption
- Reduced Application Complexity
- Automatic Secret Synchronization
- Improved Governance
- Better Separation Of Responsibilities

Applications remain portable while the platform handles secret lifecycle management.

---

# Trade-Offs

Introducing the External Secrets Operator increases platform capabilities but also introduces additional architectural complexity.

Advantages:

- Automated Secret Synchronization
- Centralized Governance
- Simplified Application Development
- Secure Identity-Based Access
- Kubernetes-Native Integration

Trade-Offs:

- Additional Kubernetes Operator
- Additional Custom Resources
- Dependency On Azure Key Vault
- Identity Configuration Complexity
- Additional Reconciliation Layer

The platform intentionally accepts these trade-offs because centralized and automated secret lifecycle management provides significantly greater security, maintainability, and operational consistency than embedding secret management directly into application workloads.

---

# Summary

The External Secrets Operator completes the secret management architecture by bridging Azure Key Vault and Kubernetes.

Azure Key Vault remains the single source of truth for sensitive information, Microsoft Entra ID authenticates workloads through Azure Workload Identity, and the External Secrets Operator continuously synchronizes centrally managed secrets into Kubernetes.

Applications consume standard Kubernetes Secrets without requiring knowledge of Azure APIs, authentication mechanisms, or secret lifecycle operations.

This architecture establishes a secure, scalable, and cloud-native approach to secret management while maintaining a clear separation between platform responsibilities and application responsibilities.
