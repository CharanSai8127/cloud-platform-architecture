# Platform Architecture

## Overview

The platform architecture is designed to provide secure secret management, workload identity, centralized governance, reusable platform capabilities, and operational consistency for applications running on Kubernetes.

Rather than treating secret management as an application responsibility, the platform provides security, storage, networking, deployment, observability, and certificate management as shared capabilities that can be consumed by multiple applications.

The objective is to allow application teams to focus on business functionality while consuming standardized platform services that satisfy security, reliability, scalability, and operational requirements.

---

# Platform Components

The platform consists of a collection of operational services that work together to provide capabilities required by application workloads.

Core platform components include:

### Deployment Platform

* ArgoCD
* ApplicationSets
* App Projects

### Secret Management Platform

* HashiCorp Vault
* Vault Agent Injector

### Storage Platform

* StorageClass
* AWS EBS CSI Driver

### Networking Platform

* NGINX Gateway
* Load Balancer
* DNS

### Certificate Management Platform

* cert-manager

### Observability Platform

* Prometheus
* Grafana
* Alerting Components

Together, these services provide a complete operational platform for application deployment and management.

---

# Platform Capability Model

The platform exposes reusable capabilities that can be consumed by multiple workloads.

---

## Security Capability

Security services provide:

* Secret Management
* Authentication
* Authorization
* Encryption
* Identity Verification

Components:

* HashiCorp Vault
* AWS KMS
* IRSA
* Kubernetes Service Accounts

---

## Storage Capability

Storage services provide:

* Dynamic Volume Provisioning
* Persistent Storage
* Stateful Workload Support

Components:

* StorageClass
* EBS CSI Driver

---

## Deployment Capability

Deployment services provide:

* GitOps Deployments
* Continuous Reconciliation
* Application Lifecycle Management

Components:

* ArgoCD
* ApplicationSets

---

## Networking Capability

Networking services provide:

* External Access
* Traffic Routing
* Service Exposure

Components:

* DNS
* Load Balancer
* NGINX Gateway

---

## Observability Capability

Observability services provide:

* Metrics Collection
* Visualization
* Alerting
* Platform Monitoring

Components:

* Prometheus
* Grafana
* Alerting Rules

---

## Certificate Management Capability

Certificate services provide:

* Certificate Issuance
* Certificate Renewal
* TLS Management

Components:

* cert-manager

---

# Operational Component Deployment Model

The platform follows a GitOps operating model where operational services and application workloads are managed through declarative configuration.

The deployment architecture is organized using:

* App Projects
* App-of-Apps
* ArgoCD Applications

This structure enables centralized governance while maintaining operational consistency across the platform.

---

## App Project

The App Project acts as the governance boundary for the platform.

It provides:

* Ownership Boundaries
* Access Control
* Source Repository Control
* Deployment Governance

All platform and application workloads operate within the governance framework defined by the project.

---

## App-of-Apps Pattern

The platform adopts the App-of-Apps pattern to manage the complete platform as a single declarative unit.

Instead of managing individual applications independently, platform services are grouped together and deployed through a higher-level parent application.

Benefits include:

* Centralized Governance
* Simplified Monitoring
* Consistent Deployment Standards
* Easier Platform Expansion

This approach allows the entire platform to be managed through a single GitOps entry point.

---

## Platform-Owned Services

Operational services are owned by the platform team because they provide capabilities required by all application workloads.

Examples include:

* Vault
* ArgoCD
* Storage Services
* Monitoring Services
* Networking Components
* Certificate Management

These services support the platform's non-functional requirements and are therefore managed centrally.

---

# Secret Management Flow

One of the primary objectives of the platform is to provide secure secret delivery without storing sensitive information within application manifests, Git repositories, or Kubernetes Secrets.

The platform achieves this through workload identity and centralized secret management.

---

## Secret Storage

Sensitive information is stored within HashiCorp Vault.

Examples include:

* Database Credentials
* API Keys
* Tokens
* Certificates

Applications never directly store or manage these secrets.

---

## Workload Authentication

When an application requires access to a secret, it authenticates using its Kubernetes Service Account.

The workload presents:

* Service Account Token
* Workload Identity

rather than static credentials.

---

## Identity Verification

Vault validates the requesting workload through Kubernetes authentication.

The verification process includes:

* JWT Validation
* Token Review
* Service Account Verification
* Vault Role Matching

This ensures that only authorized workloads can access approved secrets.

---

## Secret Injection

After successful authentication, Vault Agent Injector retrieves the secret and injects it into the pod.

The application consumes the secret directly from the runtime environment.

This eliminates the need to:

* Store Secrets In Git
* Store Secrets In Application Manifests
* Distribute Static Credentials

---

# Vault Architecture

Vault serves as the centralized secret management platform for the cluster.

The deployment is designed to maximize availability, reliability, and operational efficiency.

---

## High Availability Raft Architecture

Vault is deployed using integrated Raft storage.

This provides:

* High Availability
* Distributed Storage
* Internal Consensus
* Reduced External Dependencies

Raft enables Vault to continue operating even when individual Vault instances fail.

---

## StatefulSet Deployment

Vault is deployed using a StatefulSet.

StatefulSets provide:

* Stable Network Identity
* Persistent Storage
* Predictable Pod Management

These capabilities are required for maintaining the Vault cluster state.

---

## Persistent Storage

Vault data is persisted using dynamically provisioned volumes.

Persistent storage ensures that:

* Secret Data Survives Restarts
* Cluster State Is Preserved
* Recovery Operations Are Simplified

---

## Auto-Unseal

Vault uses AWS KMS Auto-Unseal.

Traditional manual unseal procedures introduce operational overhead and recovery complexity.

Auto-Unseal provides:

* Automated Recovery
* Faster Startup
* Reduced Operational Effort
* Consistent Unseal Operations

This improves reliability while simplifying platform operations.

---

# Application Onboarding Flow

Applications consume platform capabilities through a standardized onboarding process.

---

## Deployment Workflow

Application onboarding follows the GitOps workflow:

```text
Application Repository
        ↓
GitOps Repository
        ↓
ApplicationSet
        ↓
ArgoCD
        ↓
Kubernetes Cluster
```

Applications are deployed through declarative configuration rather than manual deployment activities.

---

## Secret Integration

Applications that require secrets must:

* Create Service Accounts
* Define Vault Roles
* Define Secret Paths

Once configured, secrets are automatically delivered through Vault authentication and injection mechanisms.

---

## Platform Capability Consumption

Applications inherit platform services including:

* Secret Management
* Storage
* Monitoring
* Networking
* Certificate Management

This reduces implementation effort while maintaining platform standards.

---

# Platform Boundaries

The platform follows a shared responsibility model between platform teams and application teams.

---

## Platform Team Responsibilities

The platform team owns:

* Kubernetes Platform
* Vault
* ArgoCD
* Storage Services
* Networking Services
* Monitoring Services
* Certificate Management
* Security Controls
* Governance Policies

The platform team provides reusable capabilities that can be consumed by application workloads.

---

## Application Team Responsibilities

Application teams own:

* Business Logic
* Application Code
* Deployment Configuration
* Resource Requirements
* Secret Consumption Requirements
* Scaling Requirements

Application teams consume platform capabilities without managing the underlying platform infrastructure.

---

# Platform Interaction Model

The platform components collaborate to provide secure application delivery and runtime operations.

A typical application request follows the flow:

```text
User
 ↓
DNS
 ↓
Load Balancer
 ↓
NGINX Gateway
 ↓
Backend Service
 ↓
Backend Pod
 ↓
Database
```

Secret retrieval occurs during workload startup and authentication rather than during every application request.

This architecture ensures that Vault does not become part of the runtime request path.

Benefits include:

* Reduced Latency
* Improved Reliability
* Lower Operational Dependency
* Simplified Runtime Architecture

Applications receive the secrets they require while remaining decoupled from the secret management system during normal request processing.

---

# Platform Architecture Summary

The platform architecture combines GitOps, centralized secret management, workload identity, reusable platform capabilities, and operational governance into a unified operating model.

By separating platform responsibilities from application responsibilities, the architecture enables secure application delivery while maintaining consistency, scalability, and operational efficiency.

The result is a platform that provides security, deployment, storage, networking, observability, and governance as reusable services that can be consumed by multiple applications without sacrificing reliability or operational simplicity.

