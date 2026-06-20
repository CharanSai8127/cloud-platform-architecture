# Security Architecture

## Overview

Security is the primary architectural driver of the platform.

As organizations adopt Kubernetes, applications require access to sensitive information such as database credentials, API keys, certificates, and tokens. Managing these secrets securely while maintaining operational efficiency becomes increasingly difficult as the number of workloads, teams, and environments grows.

The platform therefore adopts a layered security model that focuses on:

* Centralized Secret Management
* Workload Identity
* Encryption And Key Management
* Platform Isolation
* Cloud Security Integration
* Auditability
* Least Privilege Access

Rather than treating security as an application responsibility, security is implemented as a platform capability that can be consumed consistently across all workloads.

---

# Security Philosophy

The platform is designed around the principle that access to sensitive information must always be controlled, verified, and auditable.

Security controls are implemented to address:

* Unauthorized Secret Access
* Credential Exposure
* Excessive Permissions
* Misconfigured Workloads
* Operational Errors
* Cloud Access Risks

The objective is not only to protect secrets but to ensure that every access request can be authenticated, authorized, and traced.

This philosophy drives every security-related decision throughout the platform.

---

# Secret Security Model

Applications require access to sensitive information, but storing secrets directly within Kubernetes introduces operational and security concerns.

---

## Kubernetes Secret Limitations

Kubernetes Secrets are only base64 encoded by default.

While they provide a mechanism for storing sensitive values, they do not provide:

* Centralized Governance
* Secret Rotation
* Fine-Grained Access Control
* Advanced Auditability
* Secret Lifecycle Management

As platform adoption grows, these limitations become increasingly difficult to manage.

---

## Centralized Secret Management

The platform adopts HashiCorp Vault as the centralized secret management solution.

Vault becomes the authoritative source for:

* Database Credentials
* API Keys
* Tokens
* Certificates
* Sensitive Application Configuration

This provides:

* Centralized Governance
* Controlled Access
* Secret Lifecycle Management
* Improved Security Posture

Applications consume secrets without becoming responsible for storing or managing them.

---

# Workload Identity Model

The platform adopts workload identity rather than static credentials as the foundation of authentication.

Applications prove who they are before receiving access to sensitive information.

---

## Why Identity Instead Of Credentials

Traditional authentication relies on long-lived credentials.

Examples include:

* Usernames
* Passwords
* Embedded Tokens

These credentials introduce:

* Rotation Challenges
* Credential Sprawl
* Increased Exposure Risk

The platform instead uses workload identity.

Authentication is based on:

* Kubernetes Service Accounts
* Service Account Tokens
* Vault Roles
* Trust Relationships

This allows workloads to authenticate securely without relying on static credentials.

---

## Authentication Flow

When a workload requests access to a secret:

```text
Application Pod
        ↓
Service Account Token
        ↓
Vault Authentication
        ↓
Token Review
        ↓
Kubernetes API
        ↓
Identity Verification
        ↓
Vault Role Validation
        ↓
Secret Access Granted
```

This ensures that only authorized workloads can access approved secrets.

---

## Vault Authorization Model

Authentication alone is not sufficient.

Vault must also determine:

* Which workload is requesting access
* Which secrets it is allowed to access
* Whether the request complies with defined policies

This is achieved through:

* Vault Roles
* Secret Paths
* Service Account Mapping
* Access Policies

The result is a least-privilege access model for secret consumption.

---

# Encryption Architecture

Encryption protects sensitive information throughout the platform.

The objective is to provide centralized encryption and key management while minimizing operational overhead.

---

## AWS KMS

The platform uses AWS Key Management Service (KMS) as the centralized encryption service.

KMS provides:

* Centralized Key Management
* Controlled Key Access
* Key Lifecycle Management
* Auditable Key Usage

This establishes a consistent encryption model across the platform.

---

## Vault Auto-Unseal

Vault requires access to encryption keys before it can serve secrets.

A traditional Vault deployment requires manual unseal operations after restart events.

Manual unseal introduces:

* Operational Complexity
* Recovery Delays
* Human Dependency

The platform adopts KMS Auto-Unseal to eliminate these challenges.

Benefits include:

* Automated Recovery
* Reduced Operational Effort
* Faster Service Availability
* Consistent Startup Procedures

---

## Kubernetes Secret Encryption

The platform also uses centralized key management to support Kubernetes secret encryption.

This ensures that sensitive information stored within the cluster receives an additional layer of protection while maintaining operational consistency.

---

# Platform Isolation Model

Platform services and application workloads have different operational requirements and risk profiles.

The architecture therefore introduces operational isolation between platform services and application workloads.

---

## Hub Node Group

The Hub node group hosts platform-critical services:

* HashiCorp Vault
* ArgoCD
* NGINX Gateway
* Cert Manager
* Monitoring Components
* EBS CSI Controller

These services are expected to remain continuously available and provide capabilities required by application workloads.

---

## Spoke Node Group

The Spoke node group hosts:

* Application Workloads
* Databases
* Future Business Services

These workloads scale independently and have different lifecycle characteristics.

---

## Security Benefits Of Isolation

Operational separation provides:

* Reduced Blast Radius
* Predictable Resource Availability
* Controlled Platform Access
* Improved Operational Stability

This improves platform security while reducing unintended interactions between platform services and business workloads.

---

## Taints And Tolerations

Node placement controls are enforced using:

* Node Labels
* Taints
* Tolerations

These controls ensure that workloads are scheduled according to their intended purpose and operational boundaries.

This improves governance while reducing accidental workload placement.

---

# Cloud Security Integration

Cloud services expose capabilities through APIs that must be accessed securely.

The platform adopts standardized mechanisms for authentication and authorization when interacting with AWS services.

---

## IAM

IAM provides:

* Authentication
* Authorization
* Permission Boundaries

Policies define:

* Who can access resources
* Which actions can be performed
* Operational restrictions

---

## OIDC

OpenID Connect establishes trust between Kubernetes and AWS.

OIDC enables workloads to authenticate securely without requiring long-lived cloud credentials.

This forms the foundation for workload identity within AWS integrations.

---

## IAM Roles For Service Accounts (IRSA)

IRSA allows Kubernetes workloads to assume AWS roles through their Service Accounts.

Benefits include:

* Elimination Of Static Credentials
* Fine-Grained Permissions
* Improved Security
* Simplified Credential Management

Platform components that interact with AWS services use IRSA to obtain only the permissions they require.

---

## Resource Policies

Certain AWS resources require additional resource-level controls.

Examples include:

* KMS Keys
* S3 Buckets

Resource policies provide an additional authorization layer that verifies both identity and resource access requirements.

This improves governance and reduces unauthorized access.

---

# Auditability And Security Operations

Security controls must be observable.

Without visibility, security incidents become difficult to investigate and validate.

The platform therefore integrates auditing and monitoring into the security architecture.

---

## AWS CloudTrail

CloudTrail records API activity across AWS services.

Examples include:

* KMS Operations
* IAM Activity
* Service Interactions

This provides visibility into security-sensitive operations.

---

## CloudWatch Logs

CloudWatch Logs centralizes operational and security-related events.

Benefits include:

* Monitoring
* Troubleshooting
* Operational Visibility
* Security Investigation Support

---

## Amazon S3

Audit records are retained within Amazon S3 for long-term storage.

This enables:

* Historical Analysis
* Security Investigations
* Operational Traceability
* Compliance Support

---

# Secret Lifecycle Security

The platform secures the complete secret lifecycle from creation through consumption.

---

## Secret Creation

Sensitive information is stored within Vault rather than application manifests or Git repositories.

---

## Secret Authorization

Vault policies determine which workloads can access specific secret paths.

---

## Identity Verification

Workloads authenticate using Service Accounts and Kubernetes token validation.

---

## Secret Delivery

Vault Agent Injector retrieves secrets and injects them into the workload runtime environment.

---

## Secret Consumption

Applications consume secrets directly from the runtime environment without storing them within source code, Git repositories, or deployment manifests.

This reduces secret exposure while simplifying secret management.

---

# Security Failure Model

Security architectures must assume that failures and misconfigurations will occur.

The platform incorporates controls that reduce the impact of these events.

---

## Unauthorized Secret Requests

Unauthorized workloads fail authentication and are denied access to secrets.

Vault policies prevent access beyond approved boundaries.

---

## Service Account Misconfiguration

Workloads without the appropriate Service Account and Vault role mappings are unable to authenticate successfully.

---

## Vault Pod Failure

Vault operates in High Availability mode using integrated Raft storage.

Leadership is transferred automatically and secret management services remain available.

---

## KMS Service Disruption

Existing Vault instances continue operating after unseal.

New unseal operations may be delayed until connectivity to KMS is restored.

This trade-off is accepted in exchange for automated recovery and centralized key management.

---

## Node Failure

Workloads are automatically rescheduled through Kubernetes and Managed Node Groups.

Security controls remain enforced after recovery.

---

# Security Architecture Summary

The platform implements security as a shared platform capability rather than an application-specific concern.

Security is achieved through:

* Centralized Secret Management
* Workload Identity
* Centralized Encryption
* Operational Isolation
* Cloud-Native Authentication
* Least Privilege Access
* Auditability
* Automated Security Operations

Together, these controls provide a secure and scalable foundation for managing sensitive information while maintaining operational efficiency and platform reliability.

