# GitOps Architecture

## Overview

Operating a Kubernetes platform through manual deployments and configuration changes becomes increasingly difficult as the number of applications, environments, platform services, and teams grows.

The platform therefore adopts GitOps as its operational model.

GitOps establishes Git as the single source of truth for both platform services and application workloads.

All infrastructure, platform configurations, deployment definitions, and operational policies are managed declaratively through version-controlled repositories.

The objective is to improve consistency, governance, scalability, auditability, and operational reliability while reducing configuration drift and manual intervention.

---

# Why GitOps?

As platforms grow, operational complexity increases rapidly.

Common challenges include:

* Configuration Drift
* Manual Changes
* Environment Inconsistency
* Operational Errors
* Lack of Deployment Traceability
* Difficult Rollbacks

Without GitOps, cluster state gradually diverges from the intended platform state.

The platform adopts GitOps to ensure:

* Declarative Configuration
* Version Controlled Changes
* Automated Reconciliation
* Repeatable Deployments
* Auditable Operations

Git becomes the authoritative definition of platform and application state.

---

# Why ArgoCD?

GitOps requires a mechanism that continuously verifies that the actual cluster state matches the desired state defined within Git.

ArgoCD fulfills this responsibility.

ArgoCD continuously:

* Monitors Git Repositories
* Detects Configuration Changes
* Compares Desired State With Actual State
* Reconciles Drift
* Restores Platform Consistency

This ensures that the platform remains aligned with its intended configuration even when manual changes occur.

The result is a more reliable and predictable operating environment.

---

# App Project Architecture

The platform uses ArgoCD App Projects to establish governance boundaries.

An App Project defines:

* Repository Access
* Deployment Boundaries
* Namespace Restrictions
* Resource Controls
* Team Ownership

Rather than allowing unrestricted deployments across the cluster, App Projects provide centralized governance and operational consistency.

This enables platform teams to maintain control over deployment standards while supporting decentralized application ownership.

---

# App-of-Apps Architecture

Managing every platform service individually eventually creates operational complexity.

To simplify management, the platform adopts the App-of-Apps pattern.

The App-of-Apps model allows multiple ArgoCD Applications to be grouped together and managed as a single logical platform unit.

Benefits include:

* Centralized Governance
* Platform-Level Visibility
* Simplified Lifecycle Management
* Consistent Deployment Standards
* Easier Platform Expansion

The entire platform can therefore be managed through a single GitOps entry point.

---

# Separation Of Platform And Application Deployments

Operational components and application workloads have different ownership models and operational lifecycles.

The platform therefore separates deployment responsibilities into dedicated GitOps domains.

---

## Platform App-of-Apps

Platform-owned services include:

* HashiCorp Vault
* ArgoCD
* StorageClass
* AWS EBS CSI Driver
* NGINX Gateway
* cert-manager
* kube-prometheus-stack

These services provide capabilities required by all workloads running on the platform.

Because they support platform-wide non-functional requirements, they are managed centrally by the platform team.

---

## Application App-of-Apps

Application workloads are managed separately.

Examples include:

* Backend Services
* Databases
* Business Applications
* Future Workloads

This separation allows application teams to manage their workloads independently without affecting core platform services.

Benefits include:

* Independent Release Cycles
* Reduced Operational Risk
* Clear Ownership Boundaries
* Simplified Governance

---

# StorageClass As A Dedicated Platform Application

Storage is treated as a reusable platform capability rather than an application-specific dependency.

For this reason, StorageClass is deployed as an independent ArgoCD Application.

This design provides several advantages.

---

## Reusability

Multiple workloads can consume the same storage capability.

Examples include:

* HashiCorp Vault
* MySQL
* Future Stateful Workloads

---

## Dependency Management

Storage capabilities become available before dependent workloads are deployed.

This reduces deployment failures and simplifies platform operations.

---

## Platform Ownership

Storage remains a platform-managed capability rather than becoming tightly coupled to a specific workload.

This improves maintainability and operational consistency.

---

# Application Onboarding Through GitOps

Applications are onboarded through a standardized GitOps workflow.

The onboarding process follows:

```text
Application Repository
        ↓
GitOps Repository
        ↓
Application Definition
        ↓
ApplicationSet
        ↓
ArgoCD
        ↓
Kubernetes Cluster
```

Application teams define the desired state.

ArgoCD continuously reconciles that state with the cluster.

This provides a repeatable and scalable deployment model.

---

# GitOps And Secret Management

One of the primary security objectives of the platform is ensuring that Git never becomes a secret store.

Git repositories should contain:

* Configuration
* Deployment Definitions
* Policies
* References

Git repositories should not contain:

* Database Passwords
* API Keys
* Tokens
* Sensitive Credentials

The platform integrates

