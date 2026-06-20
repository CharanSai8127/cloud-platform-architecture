# Infrastructure Architecture

## Overview

The infrastructure layer provides the foundation required to support secure secret management, workload identity, operational isolation, centralized governance, and platform scalability.

While the platform inherits the core networking and Kubernetes architecture from the base EKS platform, additional infrastructure components are introduced to support encryption, authentication, auditing, and operational separation.

The objective of the infrastructure architecture is not only to provide compute and networking resources, but to establish a secure and operationally efficient foundation capable of supporting platform services and application workloads at scale.

---

# Networking Foundation

The platform is deployed within a dedicated Virtual Private Cloud (VPC) that provides network isolation and controlled communication between infrastructure resources.

The networking architecture consists of:

* Public Subnets
* Private Subnets
* Internet Gateway
* NAT Gateway
* Route Tables
* Route Table Associations

Public subnets host internet-facing infrastructure components such as load balancers and network access components.

Private subnets host:

* EKS Worker Nodes
* HashiCorp Vault
* Application Workloads
* Databases
* Platform Services

This architecture reduces exposure of critical workloads while allowing controlled outbound connectivity when required.

---

# Amazon EKS Architecture

The platform uses Amazon EKS as the Kubernetes control plane.

Operating Kubernetes control plane components introduces significant operational complexity and reliability concerns.

Amazon EKS provides:

* Managed Control Plane
* High Availability
* Automated Control Plane Management
* Reduced Operational Overhead

This allows platform teams to focus on platform capabilities rather than cluster management.

Managed Node Groups are used to host workloads and provide:

* Automated Node Recovery
* Rolling Updates
* Simplified Lifecycle Management
* Consistent Operational Practices

This improves platform reliability while reducing operational effort.

---

# Encryption And Identity Infrastructure

## AWS KMS

Sensitive platform operations require centralized encryption and key management.

The platform uses AWS KMS to provide:

* Centralized Key Management
* Controlled Key Access
* Encryption Governance
* Automated Key Operations

The KMS infrastructure is leveraged by:

* Vault Auto-Unseal
* Kubernetes Secret Encryption

This enables platform-wide encryption standards while reducing operational complexity.

---

## IAM Policies And Resource Policies

Authentication and authorization are implemented through IAM policies and resource policies.

IAM policies define:

* Who can access resources
* Which actions are permitted
* Operational boundaries

Certain AWS services require additional resource-based controls.

Examples include:

* KMS Keys
* S3 Buckets

Resource policies provide an additional trust layer that verifies both identity and resource-level access requirements.

This improves governance and reduces unauthorized access.

---

## OIDC And IRSA

Cloud integrations should not rely on static credentials.

The platform uses:

* OpenID Connect (OIDC)
* IAM Roles for Service Accounts (IRSA)

to establish trust between Kubernetes workloads and AWS services.

This enables workloads to authenticate using their Kubernetes identity while eliminating the need for long-lived cloud credentials.

IRSA is used by platform services that require controlled access to AWS APIs.

Examples include:

* Vault
* EBS CSI Driver
* Other AWS-integrated platform components

This approach improves security while simplifying credential management.

---

# Operational Isolation Architecture

One of the primary design goals of the platform is to separate platform services from application workloads.

Although both operate within the same Kubernetes cluster, they have fundamentally different operational characteristics.

Platform services provide capabilities required by applications and are expected to remain continuously available.

Application workloads are elastic and scale according to demand.

To address these differences, the platform adopts a Hub and Spoke node group architecture.

---

## Hub Node Group

The Hub node group hosts platform-critical services including:

* HashiCorp Vault
* ArgoCD
* NGINX Gateway
* Cert Manager
* Monitoring Components
* EBS CSI Controller

These services are expected to remain continuously available and provide capabilities required by application workloads.

By isolating them onto dedicated nodes, the platform reduces the likelihood of resource contention and improves platform stability.

---

## Spoke Node Group

The Spoke node group hosts:

* Application Workloads
* Databases
* Future Business Services

These workloads are expected to scale, change, and evolve more frequently than platform services.

Separating them from operational components allows workload growth without directly impacting critical platform services.

---

## Node Labels

Node labels are used to identify workload intent and node purpose.

Examples include:

* Operational Nodes
* Application Nodes

Labels provide a mechanism for workload placement and platform organization.

---

## Taints And Tolerations

Taints and tolerations are used to enforce workload placement policies.

Operational workloads are scheduled onto operational nodes.

Application workloads are scheduled onto application nodes.

This prevents accidental scheduling and improves workload isolation.

---

## Shared Platform Components

Certain platform components must operate across both node groups.

Examples include:

* Node Exporter
* EBS CSI Node Plugin

These workloads require broader scheduling permissions because they provide cluster-wide functionality.

For these scenarios, toleration policies are used to allow deployment across multiple node groups while maintaining the overall operational isolation model.

---

# Shared Infrastructure Services

## StorageClass

Persistent storage is implemented as a reusable platform capability rather than an application-specific dependency.

StorageClass is deployed independently and can be consumed by multiple workloads.

Examples include:

* HashiCorp Vault
* MySQL
* Future Stateful Applications

Treating storage as a shared platform capability improves consistency while reducing duplication.

---

## EBS CSI Driver

The EBS CSI Driver enables dynamic volume provisioning for stateful workloads.

This removes the need for manual volume management and provides a standardized storage provisioning model for the platform.

By centralizing storage operations, the platform improves operational efficiency and scalability.

---

# Audit Infrastructure

Security operations must be observable and auditable.

The platform therefore includes dedicated auditing infrastructure for monitoring security-sensitive activities.

---

## AWS CloudTrail

CloudTrail records API activity across the platform.

Examples include:

* KMS Operations
* IAM Activity
* AWS Service Interactions

This provides visibility into platform operations and security events.

---

## CloudWatch Logs

CloudWatch Logs provides centralized collection and visibility for operational and security-related events.

This improves troubleshooting, monitoring, and operational awareness.

---

## Amazon S3

Audit records are retained within Amazon S3 for long-term storage and future analysis.

This enables:

* Historical Auditing
* Security Investigations
* Compliance Support
* Operational Traceability

---

# Infrastructure Failure Model

Distributed systems are designed around the assumption that failures will occur.

The infrastructure architecture incorporates mechanisms to reduce the impact of these failures.

---

## Node Failure

Managed Node Groups continuously monitor node health.

When a node becomes unavailable, workloads are automatically rescheduled onto healthy nodes and replacement nodes are provisioned as required.

---

## Hub Node Failure

Platform services are distributed across the Hub node group.

If a node fails, operational workloads are rescheduled to healthy operational nodes, maintaining platform capabilities.

---

## Spoke Node Failure

Application workloads are rescheduled to healthy application nodes.

This minimizes application downtime and maintains service availability.

---

## Availability Zone Failure

The platform is deployed across multiple Availability Zones.

This reduces dependency on a single failure domain and improves platform resilience during infrastructure outages.

---

## Vault Pod Failure

Vault is deployed using a High Availability Raft architecture.

If a Vault pod becomes unavailable, leadership is automatically transferred and platform secret management capabilities remain operational.

---

## KMS Service Disruption

Existing Vault instances that are already unsealed continue operating normally.

However, new unseal operations may be delayed until connectivity to AWS KMS is restored.

This trade-off is accepted in exchange for automated unseal operations and centralized key management.

---

## Cluster Upgrade Events

Managed Node Groups perform rolling updates to maintain workload availability during infrastructure upgrades.

This reduces disruption while simplifying platform maintenance activities.

---

# Infrastructure Architecture Summary

The infrastructure architecture extends the foundational EKS platform by introducing centralized encryption, workload identity, operational isolation, reusable storage capabilities, audit infrastructure, and failure-resilient platform services.

Together, these components provide a secure, scalable, and operationally efficient foundation capable of supporting both platform services and application workloads while maintaining strong governance and security controls.

