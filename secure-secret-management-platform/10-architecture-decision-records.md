# Architecture Decision Records

## Overview

Every component within the platform was selected to satisfy specific architectural objectives rather than technology preferences.

The platform was designed around the following goals:

- Security
- Reliability
- Availability
- Scalability
- Operability
- Cost Efficiency
- Governance
- Maintainability

This document captures the major architectural decisions and the reasoning behind each decision.

---

# ADR-001: Amazon EKS As The Kubernetes Platform

## Decision

Use Amazon EKS as the Kubernetes platform.

## Why

The platform requires:

- High Availability
- Managed Control Plane
- Reduced Operational Overhead
- Native AWS Integration

Managing etcd, API servers, schedulers, upgrades, and control-plane recovery would significantly increase operational complexity.

Amazon EKS allows the platform team to focus on platform capabilities rather than control-plane operations.

## Outcome

- Improved Reliability
- Reduced Operational Burden
- Faster Platform Adoption

---

# ADR-002: Multi-AZ Architecture

## Decision

Deploy the platform across multiple Availability Zones.

## Why

Availability Zones represent independent failure domains.

A platform designed for production workloads should continue operating during infrastructure failures.

A Multi-AZ design improves:

- Availability
- Fault Tolerance
- Recovery Capability

## Alternative

Single AZ Deployment

### Rejected Because

A single infrastructure failure can impact the entire platform.

## Outcome

Improved platform resilience and availability.

---

# ADR-003: Hub And Spoke Node Group Architecture

## Decision

Separate operational workloads and application workloads.

## Why

Operational services:

- Vault
- ArgoCD
- Monitoring
- Gateway
- cert-manager

have different operational characteristics than applications.

Operational services:

- Must remain available
- Have predictable resource consumption
- Support the platform itself

Application workloads:

- Scale dynamically
- Change frequently
- Consume resources unpredictably

## Alternative

Single Node Group

### Rejected Because

- Resource Contention
- Increased Blast Radius
- Reduced Operational Stability

## Outcome

- Improved Reliability
- Better Capacity Planning
- Independent Scaling

---

# ADR-004: HashiCorp Vault For Secret Management

## Decision

Use HashiCorp Vault as the centralized secret management platform.

## Why

Kubernetes Secrets are only base64 encoded and do not provide complete secret lifecycle management.

The platform requires:

- Centralized Secret Governance
- Access Control
- Auditability
- Secret Rotation
- Workload Authentication

## Alternative

Kubernetes Secrets

### Rejected Because

- Limited Governance
- Limited Auditability
- Difficult Secret Rotation
- Increased Secret Exposure Risk

## Outcome

Improved security and secret governance.

---

# ADR-005: Workload Identity Instead Of Static Credentials

## Decision

Authenticate workloads through Kubernetes identities.

## Why

Static credentials create:

- Credential Sprawl
- Rotation Challenges
- Security Risks

The platform requires workloads to prove identity before accessing secrets.

Authentication is based on:

- Service Accounts
- JWT Tokens
- Vault Roles
- Token Review

## Outcome

- Improved Security
- Reduced Credential Exposure
- Better Governance

---

# ADR-006: Vault HA Raft Architecture

## Decision

Deploy Vault using HA Raft Storage.

## Why

Secret management is a critical platform capability.

A single Vault instance introduces a single point of failure.

Raft provides:

- Distributed Consensus
- Data Replication
- Leader Election
- Automated Recovery

## Alternative

Single Vault Deployment

### Rejected Because

Loss of the Vault pod results in loss of secret management capability.

## Outcome

Improved reliability and fault tolerance.

---

# ADR-007: Vault Auto-Unseal Using AWS KMS

## Decision

Use AWS KMS to automatically unseal Vault.

## Why

Manual unseal procedures:

- Increase Recovery Time
- Require Human Intervention
- Create Operational Complexity

KMS Auto-Unseal provides:

- Automated Recovery
- Consistent Startup Behavior
- Reduced Operational Overhead

## Outcome

Improved operability and recovery.

---

# ADR-008: Storage As A Platform Capability

## Decision

Deploy StorageClass as an independent platform capability.

## Why

Storage is required by:

- Vault
- MySQL
- Future Stateful Workloads

Storage should be reusable and platform-managed.

## Alternative

Storage Defined Per Application

### Rejected Because

- Duplication
- Inconsistent Configuration
- Increased Maintenance

## Outcome

Improved reusability and operational consistency.

---

# ADR-009: GitOps As The Operating Model

## Decision

Adopt GitOps as the operational model.

## Why

Manual cluster operations introduce:

- Configuration Drift
- Deployment Errors
- Inconsistent Environments

GitOps provides:

- Declarative Operations
- Version Control
- Reconciliation
- Rollback Capability

## Outcome

Improved governance and operational reliability.

---

# ADR-010: App-of-Apps Architecture

## Decision

Manage platform services through App-of-Apps.

## Why

Platform capabilities should be managed as a single logical platform unit.

Benefits include:

- Centralized Governance
- Easier Lifecycle Management
- Better Visibility

## Outcome

Improved scalability and platform management.

---

# ADR-011: IRSA For AWS Authentication

## Decision

Use IAM Roles For Service Accounts (IRSA).

## Why

Cloud integrations should not use static AWS credentials.

IRSA provides:

- Temporary Credentials
- Least Privilege Access
- Controlled Authorization

## Outcome

Improved security and governance.

---

# Architecture Decision Summary

The platform architecture is the result of deliberate decisions focused on:

- Security
- Reliability
- Scalability
- Governance
- Operability

Every major component was selected to satisfy one or more of these objectives while maintaining a balance between operational simplicity and platform capability.
