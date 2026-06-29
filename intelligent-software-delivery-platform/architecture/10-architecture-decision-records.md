# Architecture Decision Records (ADR)

## Overview

Every architecture is the result of a series of engineering decisions. Modern cloud-native platforms are not built by selecting technologies at random, but by evaluating multiple approaches, understanding their trade-offs, and choosing the solution that best satisfies the system's functional and operational requirements.

Each component introduced within this platform solves a specific engineering problem. Collectively, these decisions establish a deployment platform capable of delivering software safely, reliably, and continuously.

This document explains the architectural decisions taken throughout the project, the problems they address, the alternatives that were considered, and the trade-offs associated with each decision.

---

# ADR-01: Why Terraform?

## Problem

Cloud infrastructure consists of multiple dependent resources that must be provisioned consistently across environments.

Manual provisioning introduces:

* Configuration Drift
* Human Error
* Inconsistent Environments
* Poor Repeatability

## Decision

The infrastructure is provisioned using **Terraform**.

Infrastructure is described declaratively, allowing Azure resources to be consistently created, updated, and managed through Infrastructure as Code.

## Benefits

* Infrastructure as Code
* Declarative Provisioning
* Version Control
* Modular Design
* Environment Consistency
* Repeatability
* Automated Infrastructure Management

## Trade-Offs

* Learning Curve
* State File Management
* Planning Before Changes
* Additional Module Design Complexity

---

# ADR-02: Why Azure Kubernetes Service (AKS)?

## Problem

Managing Kubernetes control planes manually increases operational complexity and maintenance overhead.

## Decision

The platform uses **Azure Kubernetes Service (AKS)** as the managed Kubernetes platform.

Azure manages the Kubernetes control plane while engineers focus on workload deployment and platform capabilities.

## Benefits

* Managed Control Plane
* Automatic Kubernetes Upgrades
* Reduced Operational Overhead
* Azure Integration
* High Availability
* Native Kubernetes APIs

## Trade-Offs

* Cloud Provider Dependency
* Less Control Over Control Plane Components
* Azure Service Limits

---

# ADR-03: Why Azure CNI?

## Problem

Application workloads require direct integration with Azure networking.

Overlay networking introduces additional routing complexity and limits direct communication with Azure resources.

## Decision

The cluster adopts the **Azure CNI** networking model.

Every Pod receives a private IP address from the subnet associated with the AKS node pool.

## Benefits

* Native Azure Networking
* Direct VNet Communication
* Simplified Routing
* Better Azure Service Integration
* Improved Enterprise Networking

## Trade-Offs

* Higher IP Address Consumption
* Careful Subnet Planning Required
* Limited Subnet Capacity

---

# ADR-04: Why Public And Private Subnets?

## Problem

Not every Azure resource requires the same level of network exposure.

Networking infrastructure and application workloads have different security responsibilities.

## Decision

The Virtual Network is divided into:

* Public Subnet
* Private Subnet

The Public Subnet hosts networking infrastructure.

The Private Subnet hosts AKS worker nodes and application workloads.

## Benefits

* Network Segmentation
* Reduced Attack Surface
* Clear Responsibility Boundaries
* Improved Security
* Better Operational Isolation

## Trade-Offs

* Increased Network Design Complexity
* Additional Routing Configuration
* More Network Security Rules

---

# ADR-05: Why NAT Gateway?

## Problem

Applications require outbound internet connectivity while remaining inaccessible from the public internet.

## Decision

Outbound communication is performed through a **User Assigned NAT Gateway**.

The NAT Gateway performs Source Network Address Translation (SNAT) for workloads running within the private subnet.

## Benefits

* Secure Outbound Connectivity
* Predictable Public IP Address
* No Public IP On Workloads
* Better Scalability
* Enterprise Networking Model

## Trade-Offs

* Additional Azure Resource
* Additional Cost
* NAT Gateway Capacity Planning

---

# ADR-06: Why Gateway API Instead Of Ingress?

## Problem

Modern deployment strategies require advanced traffic management beyond traditional host-based and path-based routing.

## Decision

The platform adopts the **Gateway API** for ingress traffic management.

HTTPRoute is used to control request routing and weighted traffic distribution.

## Benefits

* Standard Kubernetes API
* Advanced Routing
* Weighted Traffic Splitting
* Better Progressive Delivery Integration
* Separation Of Infrastructure And Application Responsibilities
* Future-Proof Networking

## Trade-Offs

* Additional Resources
* Smaller Ecosystem Compared To Ingress
* Learning Curve

---

# ADR-07: Why Argo Rollouts Instead Of Kubernetes Deployment?

## Problem

The Kubernetes Deployment controller only manages the lifecycle of Pods.

It cannot:

* Progressively Shift Traffic
* Pause Rollouts
* Validate Metrics
* Automatically Roll Back

## Decision

The platform replaces Kubernetes Deployments with **Rollout** resources managed by Argo Rollouts.

## Benefits

* Progressive Delivery
* Canary Deployment
* Blue-Green Deployment
* Automated Promotion
* Automated Rollback
* Production Validation
* Reduced Deployment Risk

## Trade-Offs

* Additional Controller
* Additional Custom Resources
* More Operational Complexity

---

# ADR-08: Why Prometheus?

## Problem

Deployment decisions require objective production feedback rather than manual observation.

## Decision

The platform uses **Prometheus** to collect application and platform metrics.

These metrics are consumed by AnalysisTemplates during rollout.

## Benefits

* Time-Series Metrics
* Kubernetes Native
* Rich Query Language (PromQL)
* Continuous Metric Collection
* Automated Deployment Decisions

## Trade-Offs

* Storage Requirements
* Metric Retention Planning
* Query Complexity

---

# ADR-09: Why AnalysisTemplates?

## Problem

Traffic progression should depend upon application behaviour rather than elapsed time.

## Decision

The rollout executes **AnalysisTemplates** between rollout stages.

Each AnalysisTemplate evaluates production metrics against predefined success criteria.

## Benefits

* Objective Validation
* Automated Decisions
* Reduced Human Error
* Continuous Health Evaluation
* Deployment Confidence

## Trade-Offs

* Metric Threshold Tuning
* Query Design
* Additional Validation Configuration

---

# ADR-10: Why GitOps With ArgoCD?

## Problem

Manual Kubernetes deployments introduce configuration drift and reduce deployment consistency.

## Decision

The platform adopts **GitOps** using ArgoCD.

Git becomes the single source of truth for Kubernetes resources.

## Benefits

* Declarative Deployments
* Automatic Reconciliation
* Version Controlled Infrastructure
* Auditability
* Rollback Through Git History
* Consistent Environments

## Trade-Offs

* Git Repository Becomes Critical
* Additional Synchronization Layer
* GitOps Workflow Learning Curve

---

# ADR-11: Why Horizontal Pod Autoscaler (HPA)?

## Problem

Application demand changes continuously.

Static replica counts either waste infrastructure or reduce application availability.

## Decision

The platform uses the **Horizontal Pod Autoscaler** to dynamically adjust the number of application replicas according to resource utilization.

## Benefits

* Dynamic Scaling
* Improved Resource Utilization
* Better Availability
* Automatic Capacity Adjustment
* Improved Performance

## Trade-Offs

* Reactive Scaling
* Metric Dependency
* Scaling Delay

---

# ADR-12: Why Progressive Delivery?

## Problem

Traditional deployments expose every production user to the new release immediately, increasing deployment risk and blast radius.

## Decision

The platform adopts **Progressive Delivery** using Canary deployments, Gateway API, Prometheus, AnalysisTemplates, and Argo Rollouts.

Instead of relying on manual operational decisions, deployment progression depends on production feedback collected from the running application.

## Benefits

* Reduced Deployment Risk
* Progressive Traffic Exposure
* Continuous Validation
* Automated Promotion
* Automated Rollback
* Improved Reliability
* Faster Recovery

## Trade-Offs

* More Platform Components
* Higher Operational Complexity
* Greater Observability Requirements
* Additional Controller Dependencies

---

# Overall Architecture Decisions

The architectural decisions made throughout this project follow a common engineering objective.

Rather than optimizing individual technologies, every decision reduces a specific category of operational risk.

| Architectural Decision   | Primary Objective              |
| ------------------------ | ------------------------------ |
| Terraform                | Eliminate Infrastructure Drift |
| AKS                      | Managed Kubernetes Platform    |
| Azure CNI                | Native Azure Networking        |
| Public & Private Subnets | Network Isolation              |
| NAT Gateway              | Secure Outbound Connectivity   |
| Gateway API              | Advanced Traffic Management    |
| Argo Rollouts            | Progressive Delivery           |
| Prometheus               | Production Observability       |
| AnalysisTemplate         | Metrics-Based Validation       |
| ArgoCD                   | Declarative GitOps             |
| HPA                      | Dynamic Scaling                |
| Progressive Delivery     | Reliable Software Releases     |

Each decision complements the others, forming a cohesive platform architecture rather than a collection of independent technologies.

---

# Architectural Philosophy

The platform follows a consistent architectural philosophy.

* Declarative Infrastructure over Manual Provisioning.
* Automation over Manual Operations.
* Feedback-Driven Decisions over Time-Based Decisions.
* Progressive Validation over Immediate Production Exposure.
* Controller-Based Reconciliation over Imperative Execution.
* Separation of Responsibilities over Monolithic Design.
* Reliability through Continuous Observation rather than Assumptions.

These principles collectively transform software delivery into a predictable, repeatable, and resilient engineering process.

---

# Key Takeaways

Every technology introduced within this project solves a specific architectural problem. Terraform provisions consistent infrastructure, AKS provides the managed Kubernetes platform, Azure CNI integrates workloads into the Azure Virtual Network, the Gateway API enables advanced traffic management, Argo Rollouts orchestrates Progressive Delivery, Prometheus supplies production telemetry, AnalysisTemplates validate application health, ArgoCD ensures declarative reconciliation, and the Horizontal Pod Autoscaler dynamically adapts application capacity.

Together, these decisions create a platform that continuously reduces operational risk while improving security, automation, scalability, and reliability. Rather than functioning as isolated tools, each component contributes to a unified architecture where software releases are progressively introduced, continuously validated, and automatically promoted or rolled back based on real production behaviour.

