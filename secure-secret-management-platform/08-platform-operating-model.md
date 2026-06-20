# Platform Operating Model

## Overview

Building a platform is not only about deploying infrastructure, Kubernetes clusters, or platform services.

The long-term success of a platform depends on how it is operated, governed, consumed, and evolved over time.

The platform is designed as a shared service that provides reusable capabilities to application teams while maintaining centralized governance, security standards, operational consistency, and platform reliability.

Rather than operating applications directly, the platform team provides capabilities that applications consume to satisfy their functional and non-functional requirements.

This operating model enables platform scalability without creating operational bottlenecks.

---

# Operating Model Philosophy

The platform is treated as a product rather than a collection of infrastructure components.

The primary responsibility of the platform is to provide capabilities that can be consumed by multiple applications.

Examples include:

* Secret Management
* Storage
* Networking
* Certificate Management
* Observability
* GitOps Deployments

Applications consume these capabilities without becoming responsible for managing the underlying platform infrastructure.

This allows application teams to focus on business requirements while the platform team focuses on platform capabilities.

---

# Shared Responsibility Model

The platform follows a shared responsibility model between platform teams and application teams.

Neither team operates independently.

The platform team provides the foundation required to run applications, while application teams consume those capabilities and define workload-specific requirements.

This collaboration allows the platform to evolve alongside application needs while maintaining governance and operational consistency.

---

# Platform Team Responsibilities

The platform team owns the capabilities that support application workloads.

These responsibilities include:

* Kubernetes Platform
* Amazon EKS
* HashiCorp Vault
* ArgoCD
* Storage Services
* Networking Services
* Monitoring Services
* Certificate Management
* Security Controls
* Governance Policies
* Cloud Integrations

The platform team is responsible for ensuring that these services remain:

* Available
* Secure
* Reliable
* Scalable
* Operationally Consistent

The platform team focuses on enabling application delivery rather than managing application business logic.

---

# Application Team Responsibilities

Application teams own the workloads that deliver business functionality.

Responsibilities include:

* Application Code
* Business Logic
* Deployment Configuration
* Resource Requirements
* Secret Consumption Requirements
* Scaling Requirements
* Application Availability Requirements

Application teams are responsible for defining how their workloads consume platform capabilities.

This allows application teams to remain autonomous while operating within platform standards.

---

# Platform Consumption Model

Applications interact with the platform through reusable platform services.

The platform provides standardized capabilities that can be consumed as required.

---

## Secret Management

Applications requiring sensitive information consume secrets through Vault.

Examples include:

* Database Credentials
* API Keys
* Tokens
* Certificates

Applications authenticate using workload identity and receive secrets through platform-managed mechanisms.

---

## Storage

Applications requiring persistence consume platform-managed storage.

Examples include:

* Databases
* Stateful Applications
* Persistent Services

Storage provisioning is standardized through platform-managed StorageClasses.

---

## Networking

Applications requiring external access consume platform networking services.

Examples include:

* DNS
* Load Balancers
* Gateway Services

This provides a consistent traffic management model across workloads.

---

## Observability

Applications inherit platform observability capabilities.

Examples include:

* Metrics Collection
* Dashboards
* Alerting
* Monitoring

This reduces duplication while improving operational visibility.

---

## Certificate Management

Applications consume platform-managed TLS capabilities through cert-manager.

This simplifies certificate issuance and lifecycle management.

---

# Platform Governance Model

As platform adoption grows, governance becomes increasingly important.

The platform implements governance through:

* GitOps
* App Projects
* Service Accounts
* Vault Policies
* IAM Policies
* IRSA
* Namespace Boundaries

These controls ensure that platform standards remain consistent while supporting decentralized application ownership.

Governance is designed to enable teams rather than restrict them.

---

# Platform Evolution Model

The platform is designed to evolve over time.

New requirements, workloads, and operational capabilities will emerge as adoption increases.

The architecture supports incremental expansion through reusable platform capabilities.

Examples of future platform capabilities may include:

* External Secret Integrations
* Progressive Delivery
* Service Mesh
* Multi-Cluster Deployments
* Advanced Policy Enforcement
* Additional Observability Services

Because platform services are managed independently from applications, new capabilities can be introduced without requiring significant application changes.

This allows the platform to grow alongside organizational requirements.

---

# Operational Workflow

The platform provides a standardized workflow for onboarding and operating applications.

---

## New Application Onboarding

When a new application is introduced:

```text
Application Team
        ↓
Defines Application Requirements
        ↓
Consumes Platform Capabilities
        ↓
Creates Deployment Definitions
        ↓
GitOps Deployment
        ↓
Application Becomes Operational
```

The platform provides the required capabilities while the application team remains responsible for workload behavior.

---

## Secret Consumption Workflow

When an application requires access to a secret:

```text
Application
        ↓
Service Account Identity
        ↓
Vault Authentication
        ↓
Policy Validation
        ↓
Secret Delivery
```

The application consumes the secret without becoming responsible for secret storage or distribution.

---

## Storage Consumption Workflow

When persistent storage is required:

```text
Application
        ↓
Persistent Volume Claim
        ↓
StorageClass
        ↓
Dynamic Provisioning
        ↓
Persistent Storage
```

The platform provides storage capabilities while maintaining centralized governance.

---

# Platform Ownership Boundaries

Clear ownership boundaries are essential for operational efficiency.

The platform establishes ownership boundaries that reduce ambiguity while encouraging collaboration.

---

## Platform-Owned Capabilities

The platform team owns:

* Infrastructure
* Security Services
* Networking
* Storage
* Observability
* GitOps Services
* Certificate Management
* Cloud Integrations

These services are treated as shared capabilities consumed by multiple workloads.

---

## Application-Owned Capabilities

Application teams own:

* Application Code
* Business Logic
* Application Configuration
* Resource Requirements
* Scaling Requirements
* Application Lifecycle

Application teams determine how their workloads consume platform capabilities.

---

## Shared Responsibilities

Certain responsibilities require collaboration between both teams.

Examples include:

* Secret Access Requirements
* Resource Consumption
* Reliability Objectives
* Scaling Expectations
* Operational Standards

The platform team provides the capability.

The application team defines how the capability is consumed.

This creates a decentralized operating model while maintaining centralized governance.

---

# Operating Model Summary

The platform operating model is designed around the concept of shared responsibility and reusable platform capabilities.

The platform team provides:

* Security
* Storage
* Networking
* Observability
* GitOps
* Governance

Application teams consume these services to deliver business functionality.

This approach allows the platform to scale across multiple applications and teams while maintaining security, consistency, reliability, and operational efficiency.

The result is a platform that functions as a reusable product, enabling application teams to move faster while remaining aligned with organizational standards and platform governance.

