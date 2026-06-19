# Infrastructure Architecture

## Overview

A cloud-native platform is only as reliable as the infrastructure it runs on.

Before applications, GitOps workflows, observability stacks, security controls, or operational tooling can be introduced, the underlying infrastructure must provide a secure, scalable, reliable, and highly available foundation capable of supporting future platform growth.

The objective of the infrastructure layer is not simply to provision resources. It is to establish an environment capable of hosting applications, platform services, monitoring systems, security controls, and future operational capabilities without requiring architectural redesign.

The infrastructure architecture was designed around the architectural drivers discussed previously:

* Availability
* Reliability
* Scalability
* Security
* Performance
* Operability
* Cost Efficiency

Every infrastructure decision described in this section can be traced back to one or more of these drivers.

---

# Network Isolation Strategy

## Why a VPC?

The first architectural decision was establishing a dedicated network boundary.

Cloud resources are provisioned within a shared public cloud environment. Without network isolation, workloads lack clear security boundaries and operational control.

A Virtual Private Cloud (VPC) provides an isolated networking environment where platform resources can be deployed and managed independently from other workloads running within AWS.

The VPC serves as the foundational security boundary for the platform.

### Architectural Benefits

| Driver          | Contribution                    |
| --------------- | ------------------------------- |
| Security        | Isolated network boundary       |
| Operability     | Centralized network management  |
| Scalability     | Supports future platform growth |
| Maintainability | Consistent network architecture |

---

# Modular Infrastructure Design

## Why Separate Terraform Modules?

The platform was designed using dedicated Terraform modules for:

* VPC
* EKS Control Plane
* Managed Node Groups

Rather than managing all infrastructure within a single Terraform configuration, infrastructure responsibilities were separated according to their purpose.

This approach enables independent lifecycle management while reducing coupling between infrastructure components.

### Architectural Benefits

| Driver          | Contribution                        |
| --------------- | ----------------------------------- |
| Operability     | Easier maintenance                  |
| Scalability     | Independent expansion of components |
| Maintainability | Clear separation of concerns        |
| Reliability     | Reduced configuration complexity    |

---

# Public and Private Subnet Strategy

## Why Public Subnets?

Public subnets host infrastructure components that require internet connectivity.

Examples include:

* Internet Gateway
* NAT Gateway
* External Load Balancers

These resources act as controlled entry and exit points for platform traffic.

## Why Private Subnets?

Private subnets host:

* Kubernetes Worker Nodes
* Application Workloads
* Platform Services
* Monitoring Components

The objective is to ensure workloads are not directly exposed to the public internet.

This significantly reduces the attack surface while maintaining operational flexibility.

### Architectural Benefits

| Driver      | Contribution                      |
| ----------- | --------------------------------- |
| Security    | Reduced public exposure           |
| Reliability | Isolated workload execution       |
| Operability | Clear network boundaries          |
| Scalability | Supports future platform services |

---

# Controlled Internet Access Strategy

## Why NAT Gateway?

Applications and platform components deployed within private subnets still require outbound access.

Examples include:

* Downloading container images
* Accessing AWS APIs
* Provisioning persistent storage
* Exporting metrics
* Communicating with managed cloud services

The challenge is providing outbound connectivity without exposing workloads directly to the internet.

A NAT Gateway satisfies this requirement by allowing private resources to initiate outbound connections while preventing inbound internet access.

### Architectural Benefits

| Driver      | Contribution                      |
| ----------- | --------------------------------- |
| Security    | No direct inbound access          |
| Reliability | Access to required cloud services |
| Operability | Simplified network management     |
| Performance | Consistent outbound connectivity  |

### Trade-Off

NAT Gateways introduce additional infrastructure cost but provide a significantly stronger security posture.

---

# High Availability Strategy

## Why Multi-AZ Design?

Infrastructure failures are inevitable.

Availability Zones represent independent failure domains within a region.

If all workloads are deployed within a single Availability Zone, an AZ failure can result in a complete platform outage.

To reduce this risk, infrastructure resources were distributed across multiple Availability Zones.

This ensures that workload execution can continue even if a single Availability Zone becomes unavailable.

### Architectural Benefits

| Driver       | Contribution                        |
| ------------ | ----------------------------------- |
| Availability | Fault tolerance across AZs          |
| Reliability  | Reduced failure impact              |
| Scalability  | Distributed infrastructure capacity |
| Operability  | Safer maintenance operations        |

### Trade-Off

Multi-AZ architectures increase networking complexity and may introduce additional cross-AZ traffic costs.

---

# Managed Kubernetes Strategy

## Why Amazon EKS?

A Kubernetes platform requires a highly available control plane responsible for:

* API Server
* Scheduler
* Controller Manager
* etcd

Operating and maintaining these components introduces significant operational overhead.

The objective of the platform team is to provide application capabilities rather than manage Kubernetes internals.

Amazon EKS was selected to delegate control plane operations to AWS while allowing the platform team to focus on higher-level platform concerns.

### Architectural Benefits

| Driver       | Contribution                         |
| ------------ | ------------------------------------ |
| Availability | Multi-AZ control plane               |
| Reliability  | AWS-managed control plane operations |
| Scalability  | Managed Kubernetes platform          |
| Operability  | Reduced operational burden           |

### Trade-Off

The platform gains operational simplicity at the expense of reduced control over control plane internals.

---

# Compute Management Strategy

## Why Managed Node Groups?

Worker nodes provide the compute capacity required to run platform services and applications.

Managing worker node lifecycle manually introduces operational complexity around:

* Node replacement
* Upgrades
* Health monitoring
* Scaling

Managed Node Groups automate these responsibilities and integrate directly with EKS.

### Architectural Benefits

| Driver       | Contribution                 |
| ------------ | ---------------------------- |
| Reliability  | Automated node replacement   |
| Availability | Managed upgrade processes    |
| Scalability  | Simplified cluster expansion |
| Operability  | Reduced management effort    |

### Trade-Off

Managed Node Groups provide less customization than self-managed node pools.

---

# Upgrade Strategy

## Why Rolling Updates?

Infrastructure upgrades are unavoidable.

The objective is to perform maintenance activities without causing application downtime.

Managed Node Groups support rolling updates, allowing nodes to be replaced gradually while workloads continue running on remaining healthy nodes.

This reduces operational risk and supports platform availability objectives.

### Architectural Benefits

| Driver       | Contribution                     |
| ------------ | -------------------------------- |
| Availability | Reduced downtime during upgrades |
| Reliability  | Controlled node replacement      |
| Operability  | Safer maintenance process        |

---

# Infrastructure Architecture Summary

The infrastructure architecture was designed to establish a secure, scalable, highly available, and operationally manageable foundation for the platform.

Rather than optimizing for rapid deployment, the architecture prioritizes long-term sustainability by focusing on network isolation, fault tolerance, managed services, operational simplicity, and future platform growth.

The result is an infrastructure foundation capable of supporting applications, platform services, observability systems, security controls, and future operational requirements while maintaining alignment with the platform's architectural objectives.

