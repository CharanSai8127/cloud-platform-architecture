# Architecture Drivers

## Overview

The platform was not designed around a specific technology or product. It was designed around a set of architectural challenges that emerge when multiple applications, platform services, and teams operate within the same Kubernetes environment.

As platform adoption increases, requirements around security, governance, secret management, operational consistency, and scalability become increasingly important.

The architecture was therefore designed to address these challenges while maintaining the platform's objectives of reliability, security, operability, and scalability.

The following architectural drivers influenced the design decisions implemented throughout the platform.

---

# Secure Secret Management

Applications require access to sensitive information such as:

* Database Credentials
* API Keys
* Tokens
* Certificates

By default, Kubernetes Secrets are only base64 encoded and should not be considered a complete secret management solution.

As the platform grows, storing sensitive information directly within manifests or relying on Kubernetes Secrets introduces operational and security risks.

Challenges include:

* Secret Exposure
* Credential Sprawl
* Manual Secret Management
* Difficult Secret Rotation
* Lack of Centralized Governance

The platform therefore requires a centralized mechanism for storing, managing, and distributing sensitive information.

This requirement ultimately drives the adoption of HashiCorp Vault as the platform's centralized secret management solution.

---

# Identity-Based Authentication

The platform must ensure that only authorized workloads can access the secrets they require.

Simply storing secrets centrally does not solve the problem of access control.

The platform requires a mechanism to verify:

* Who is requesting the secret
* Whether the request originates from the intended cluster
* Whether the workload is authorized to consume the requested secret

To satisfy these requirements, workloads authenticate through Kubernetes identities rather than static credentials.

Authentication is performed using:

* Kubernetes Service Accounts
* Service Account Tokens
* Kubernetes Token Review
* Vault Roles and Policies

This approach reduces the risks associated with long-lived credentials while providing a stronger trust model between workloads and the secret management platform.

---

# Centralized Encryption And Key Management

Encryption is a fundamental security requirement for platform services that manage sensitive information.

The challenge is not simply encrypting data, but managing encryption consistently across the platform.

Without centralized key management, encryption strategies become fragmented and difficult to govern.

The platform therefore requires:

* Centralized Key Management
* Controlled Key Access
* Automated Recovery Mechanisms
* Consistent Encryption Standards

AWS KMS is used to satisfy these requirements and serves as the centralized key management system for both:

* Vault Auto-Unseal
* Kubernetes Secret Encryption

This creates a consistent security model while reducing operational complexity.

---

# Operational Isolation

Platform services and application workloads have fundamentally different operational characteristics.

Platform services such as:

* HashiCorp Vault
* ArgoCD
* NGINX Gateway
* Cert Manager
* Monitoring Components

provide capabilities required by the rest of the platform and are expected to remain continuously available.

Application workloads are typically:

* Elastic
* Frequently Updated
* Demand Driven
* Scaled Based On Traffic

Running both categories of workloads without operational boundaries can introduce resource contention and reduce platform stability.

The architecture therefore requires a mechanism to separate platform services from application workloads while still allowing them to operate within the same cluster.

This requirement drives the adoption of dedicated operational and application node groups.

---

# Platform Reusability

The platform is expected to support multiple workloads over time.

Capabilities should therefore be designed as reusable services rather than application-specific implementations.

Examples include:

* Storage
* Secret Management
* Monitoring
* Ingress
* Certificate Management

Reusable platform capabilities reduce duplication and simplify future application onboarding.

This architectural driver influenced decisions such as deploying StorageClass as a dedicated platform capability that can be consumed by multiple stateful workloads rather than being tightly coupled to a single application.

---

# Governance And Cloud Integrations

Modern platforms rely heavily on managed cloud services.

Examples include:

* AWS KMS
* EBS CSI Driver
* CloudTrail
* CloudWatch
* IAM
* IRSA

These services expose functionality through APIs that require controlled access and governance.

Without a standardized integration model, permissions become difficult to manage and operational consistency becomes increasingly difficult to maintain.

The platform therefore requires:

* Standardized Cloud Access
* Controlled Permissions
* Centralized Governance
* Consistent Integration Patterns

This requirement drives the use of IAM policies, IRSA, resource policies, and platform-managed cloud integrations.

---

# Auditability And Observability

Security controls must be observable.

Without visibility into security operations, it becomes difficult to investigate incidents, validate access patterns, or verify compliance requirements.

The platform therefore requires mechanisms to monitor and audit security-related activities.

Examples include:

* Key Usage
* Authentication Events
* Platform Operations
* Access Patterns

To support these requirements, key management activities are integrated with:

* AWS CloudTrail
* CloudWatch Logs
* Centralized Log Storage

This provides visibility into platform security operations while improving governance and traceability.

---

# Platform Scalability

Scalability extends beyond application traffic.

As platform adoption increases, the platform must support growth in:

* Applications
* Teams
* Secrets
* Stateful Workloads
* Platform Services
* Cloud Integrations

The architecture must therefore provide mechanisms that allow these capabilities to grow without introducing significant operational overhead.

The objective is to ensure that platform services remain manageable and consistent as adoption increases.

This requirement influences decisions around centralized secret management, reusable platform capabilities, GitOps automation, and operational standardization.

---

# Architecture Drivers Summary

The architecture was designed to address a collection of operational, security, governance, and scalability challenges commonly encountered within modern cloud-native platforms.

The primary architectural drivers include:

* Secure Secret Management
* Identity-Based Authentication
* Centralized Encryption And Key Management
* Operational Isolation
* Platform Reusability
* Governance And Cloud Integrations
* Auditability And Observability
* Platform Scalability

These drivers collectively shaped the platform architecture and directly influenced the design decisions implemented throughout the solution.

