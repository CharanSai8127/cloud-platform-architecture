# Design Decisions and Trade-Offs

## Overview

Every architectural decision introduces both benefits and trade-offs. The objective was not to build the most complex platform possible, but to design a platform that balances operational simplicity, security, scalability, reliability, and cost efficiency.

The following decisions were made based on the non-functional requirements established during the design phase.

---

# 1. Amazon EKS vs Self-Managed Kubernetes

## Decision

Use Amazon EKS as the Kubernetes control plane.

## Why

Managing a Kubernetes control plane introduces operational complexity around:

* etcd management
* Control plane upgrades
* High availability
* Backup and recovery
* Security patching

By adopting EKS, these responsibilities are delegated to AWS, allowing platform efforts to focus on application delivery and operational excellence.

## Benefits

* Reduced operational overhead
* Managed control plane availability
* Simplified upgrades
* Faster platform delivery

## Trade-Offs

* Less control over control plane components
* Dependence on AWS-managed services
* Additional managed service costs

---

# 2. Private Worker Nodes vs Public Worker Nodes

## Decision

Deploy worker nodes within private subnets.

## Why

Application workloads should not be directly exposed to the public internet.

The platform should enforce a security-first approach where external traffic is intentionally controlled rather than implicitly allowed.

## Benefits

* Reduced attack surface
* Improved security posture
* Better workload isolation
* Controlled network access patterns

## Trade-Offs

* Requires NAT Gateway for outbound connectivity
* Additional networking complexity
* Additional infrastructure cost

---

# 3. Multi-AZ Deployment vs Single-AZ Deployment

## Decision

Distribute worker nodes across multiple Availability Zones.

## Why

Availability was identified as a critical platform requirement.

The platform should tolerate failures affecting a single Availability Zone without causing application outages.

## Benefits

* Improved fault tolerance
* Increased workload availability
* Reduced infrastructure risk

## Trade-Offs

* Increased networking complexity
* Additional cross-AZ traffic costs
* More complex operational planning

---

# 4. Managed Node Groups vs Self-Managed Nodes

## Decision

Use Amazon EKS Managed Node Groups.

## Why

Node lifecycle management is a repetitive operational activity that can be automated through managed services.

## Benefits

* Simplified upgrades
* Reduced operational burden
* Integrated scaling support
* Standardized node management

## Trade-Offs

* Less customization flexibility
* AWS-specific implementation
* Reduced control compared to self-managed nodes

---

# 5. Platform Governance vs Developer Freedom

## Decision

Implement governance controls through Resource Quotas, Limit Ranges, and Namespace Security Policies.

## Why

Without governance controls, workloads can consume excessive resources and negatively impact cluster stability.

The platform should establish operational boundaries while still enabling developer productivity.

## Benefits

* Predictable resource utilization
* Improved platform stability
* Reduced risk of resource exhaustion
* Consistent operational standards

## Trade-Offs

* Additional administrative overhead
* Reduced deployment flexibility
* Increased onboarding requirements

---

# 6. Horizontal Pod Autoscaling vs Static Capacity

## Decision

Implement Horizontal Pod Autoscalers for application workloads.

## Why

Workload demand changes over time and should not require manual intervention for every scaling event.

## Benefits

* Dynamic workload scaling
* Improved resource efficiency
* Better user experience during traffic spikes

## Trade-Offs

* Additional monitoring dependencies
* Scaling configuration complexity
* Potential resource fluctuations

---

# 7. Pod Disruption Budgets vs Operational Flexibility

## Decision

Protect critical workloads using Pod Disruption Budgets.

## Why

Maintenance operations should not accidentally reduce application availability below acceptable thresholds.

## Benefits

* Improved reliability
* Safer node maintenance
* Reduced deployment risk

## Trade-Offs

* Slower maintenance operations
* Potential scheduling constraints
* Additional operational planning

---

# 8. Infrastructure as Code vs Manual Provisioning

## Decision

Provision infrastructure using Terraform modules.

## Why

Manual provisioning creates inconsistency, configuration drift, and operational risk.

Infrastructure should be repeatable, auditable, and version controlled.

## Benefits

* Consistent deployments
* Version-controlled infrastructure
* Improved maintainability
* Reduced human error

## Trade-Offs

* Learning curve
* State management complexity
* Additional tooling requirements

---

# 9. Cost Optimization Through CAST AI

## Decision

Use CAST AI to continuously optimize cluster compute resources.

## Why

Initial cluster sizing is often based on assumptions and can result in underutilized resources.

The platform should continuously adapt infrastructure allocation to workload requirements.

## Benefits

* Improved resource utilization
* Reduced cloud spending
* Automated infrastructure optimization

## Trade-Offs

* Dependency on an external optimization platform
* Reduced predictability of node composition
* Additional operational integration

---

# 10. Platform-First Design vs Application-First Design

## Decision

Design the platform before optimizing individual applications.

## Why

Applications evolve over time, but platform capabilities should remain consistent and reusable across workloads.

The platform should provide standardized deployment, governance, scalability, and operational patterns that all applications can consume.

## Benefits

* Reusable platform capabilities
* Consistent operational model
* Simplified application onboarding
* Long-term scalability

## Trade-Offs

* Higher initial platform investment
* Additional architectural planning
* More upfront engineering effort

---

# Decision Summary

| Decision                   | Primary Benefit              | Primary Trade-Off       |
| -------------------------- | ---------------------------- | ----------------------- |
| Amazon EKS                 | Reduced operational overhead | Less control            |
| Private Worker Nodes       | Improved security            | NAT dependency          |
| Multi-AZ Deployment        | Higher availability          | Increased cost          |
| Managed Node Groups        | Easier operations            | Less flexibility        |
| Platform Governance        | Cluster stability            | Reduced freedom         |
| Horizontal Pod Autoscaling | Dynamic scaling              | Additional complexity   |
| Pod Disruption Budgets     | Improved reliability         | Maintenance constraints |
| Infrastructure as Code     | Consistency                  | Tooling complexity      |
| CAST AI Optimization       | Cost reduction               | External dependency     |
| Platform-First Design      | Reusability                  | Higher upfront effort   |

These decisions collectively shaped the AWS EKS Platform and ensured that the architecture aligned with the platform's goals of security, scalability, reliability, governance, and operational efficiency.

