# GitOps And Platform Operations

## Overview

Modern cloud-native platforms are no longer managed through manual deployments or imperative operational procedures. As infrastructure and application complexity continue to grow, maintaining consistency across environments becomes increasingly difficult. Manual deployments introduce configuration drift, inconsistent environments, deployment failures, and operational overhead.

GitOps addresses these challenges by treating Git repositories as the single source of truth for both infrastructure and application configuration. Every platform capability, Kubernetes resource, and application deployment is declared within version-controlled repositories, allowing the cluster to continuously reconcile itself toward the desired state.

This project adopts GitOps using ArgoCD to automate deployment, continuously reconcile platform resources, eliminate configuration drift, and establish a declarative operational model for the entire AKS platform.

---

# Why GitOps?

Traditional deployment models rely on engineers manually applying infrastructure and Kubernetes resources.

Examples include:

- kubectl apply
- Helm Install
- Manual Configuration Changes
- Direct Cluster Modifications

Although these approaches work for small environments, they become increasingly difficult to manage as platforms grow.

Problems include:

- Configuration Drift
- Manual Errors
- Inconsistent Environments
- Lack Of Version Control
- Difficult Rollbacks
- Reduced Auditability

GitOps eliminates these problems by ensuring that Git becomes the single source of truth.

---

# Git As The Source Of Truth

Every infrastructure and application configuration is maintained declaratively within Git repositories.

Rather than modifying the cluster directly, engineers modify Git.

Git therefore becomes responsible for:

- Desired Configuration
- Version History
- Change Tracking
- Collaboration
- Rollback
- Auditing

The Kubernetes cluster continuously synchronizes itself with the desired state defined in Git.

This creates a predictable and repeatable deployment model.

---

# Why ArgoCD?

Git alone stores configuration.

It does not continuously apply changes to the Kubernetes cluster.

ArgoCD extends Kubernetes' reconciliation model by continuously comparing the cluster's actual state with the desired state stored in Git.

Whenever differences are detected, ArgoCD automatically reconciles the cluster.

This establishes continuous synchronization without manual intervention.

---

# Continuous Reconciliation

Kubernetes is fundamentally built on reconciliation.

Native controllers continuously reconcile resources such as:

- Deployments
- StatefulSets
- Services
- ReplicaSets

GitOps extends this concept.

Instead of reconciling only Kubernetes resources, ArgoCD reconciles the desired platform configuration stored within Git.

The reconciliation process becomes:

```text
Git Repository
        │
        ▼
ArgoCD
        │
        ▼
Desired State
        │
        ▼
Kubernetes Cluster
        │
        ▼
Current State
```

Whenever differences exist, ArgoCD automatically restores the desired configuration.

---

# Platform And Application Separation

One of the primary architectural principles adopted in this project is separating platform capabilities from application workloads.

Although both are deployed onto the same Kubernetes cluster, they possess different operational responsibilities.

## Platform Components

Platform components provide reusable capabilities consumed by multiple applications.

Examples include:

- External Secrets Operator
- Gateway API
- Monitoring Components
- Platform Services

These components evolve independently of application releases.

---

## Application Components

Application workloads represent business functionality.

Examples include:

- .NET Application
- Services
- Deployments
- Application Configuration

Applications should not own platform infrastructure.

Instead, they consume platform capabilities exposed by the platform team.

This separation simplifies lifecycle management and reduces operational coupling.

---

# Declarative Platform Management

Every Kubernetes resource is described declaratively.

Examples include:

- Namespace
- Deployment
- Service
- Service Account
- ExternalSecret
- ClusterSecretStore
- Gateway Resources

Rather than manually configuring the cluster, the desired state is expressed through declarative manifests.

The cluster continuously converges toward this desired state.

---

# Application Lifecycle

The deployment lifecycle becomes entirely Git-driven.

```text
Developer
        │
        ▼
Git Commit
        │
        ▼
Git Repository
        │
        ▼
ArgoCD
        │
        ▼
Kubernetes Cluster
        │
        ▼
Running Application
```

Every deployment follows the same process.

No direct interaction with the Kubernetes cluster is required.

---

# Resource Ordering

Enterprise Kubernetes platforms frequently contain dependencies between resources.

Examples include:

- Custom Resource Definitions
- Kubernetes Operators
- Custom Resources
- Application Workloads

Certain resources cannot be created until their dependencies already exist.

ArgoCD provides Sync Waves to establish deterministic resource ordering.

For example:

```text
Custom Resource Definitions
        │
        ▼
Operators
        │
        ▼
Custom Resources
        │
        ▼
Application Workloads
```

This ensures Kubernetes receives resources in the correct sequence.

---

# Sync Waves And Runtime Readiness

Although Sync Waves guarantee resource ordering, they do not guarantee runtime readiness.

This distinction is important.

Sync Waves answer:

**Which resource should Kubernetes create first?**

They do **not** answer:

**Has the controller become operational and started reconciling resources?**

For example, an operator may still be:

- Starting
- Synchronizing Informer Caches
- Performing Leader Election
- Registering Admission Webhooks
- Initializing Controllers

while ArgoCD has already completed the Deployment.

This means:

**Resource Ordering ≠ Controller Readiness**

Additional synchronization mechanisms may therefore be required for operator-based platforms.

This represents one of the key architectural trade-offs encountered during implementation.

---

# Drift Detection

Configuration drift occurs whenever the running cluster differs from the desired configuration stored within Git.

Examples include:

- Manual Resource Modification
- Accidental Deletion
- Unauthorized Changes
- Failed Updates

ArgoCD continuously detects these differences and restores the cluster to its intended state.

This significantly improves operational consistency.

---

# Rollback Strategy

Every platform change is version controlled.

If a deployment introduces unexpected behavior, recovery simply involves reverting the corresponding Git commit.

ArgoCD automatically reconciles the cluster back to the previous stable configuration.

Rollback therefore becomes:

```text
Git Revert
        │
        ▼
ArgoCD Reconciliation
        │
        ▼
Previous Stable State
```

No manual Kubernetes operations are required.

---

# Advantages

Adopting GitOps provides several operational benefits.

These include:

- Declarative Deployments
- Continuous Reconciliation
- Version Control
- Drift Detection
- Repeatable Deployments
- Simplified Rollback
- Improved Auditability
- Reduced Operational Errors
- Consistent Environments

Platform operations become predictable, repeatable, and fully traceable.

---

# Trade-Offs

GitOps significantly improves platform operations but introduces additional architectural complexity.

Advantages:

- Automated Synchronization
- Declarative Management
- Strong Auditability
- Reduced Manual Operations
- Improved Operational Consistency

Trade-Offs:

- Additional Platform Components
- Git Repository Dependency
- Reconciliation Learning Curve
- Operator Readiness Considerations
- Increased Initial Platform Design

The project intentionally accepts these trade-offs because declarative platform management provides substantially greater operational reliability than imperative deployment models.

---

# Summary

GitOps establishes the operational model for the entire AKS platform.

Git becomes the single source of truth, ArgoCD continuously reconciles the Kubernetes cluster, and every infrastructure or application change follows a controlled, versioned deployment workflow.

By separating platform capabilities from application workloads, adopting declarative resource management, continuously detecting configuration drift, and automating reconciliation, the platform achieves consistent, repeatable, and enterprise-grade operations.

One of the key lessons from this implementation is that while ArgoCD Sync Waves provide deterministic resource ordering, they do not guarantee controller runtime readiness. Understanding this distinction between **resource creation** and **controller readiness** is essential when designing reliable operator-based Kubernetes platforms.
