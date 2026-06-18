# Non-Functional Requirements

## Overview

Before selecting technologies, designing infrastructure, or deploying applications, a set of non-functional requirements was established to define how the platform should operate under real-world conditions.

These requirements influenced every architectural and operational decision made throughout the platform design process.

---

# 1. Availability

The platform should remain operational even when individual infrastructure components experience failures.

### Requirements

* Eliminate single points of failure where practical.
* Distribute workloads across multiple Availability Zones.
* Support workload continuity during node maintenance and upgrades.
* Minimize application downtime during operational events.

### Architectural Impact

* Multi-AZ deployment strategy.
* Managed Kubernetes control plane.
* Pod Disruption Budgets (PDBs).
* Redundant worker node placement.

---

# 2. Scalability

The platform should be capable of handling changing workload demands without requiring manual scaling activities.

### Requirements

* Support horizontal application scaling.
* Support infrastructure growth based on workload demand.
* Accommodate future platform expansion.

### Architectural Impact

* Horizontal Pod Autoscalers (HPA).
* Managed Node Groups.
* Dynamic infrastructure scaling.
* Kubernetes-native scheduling.

---

# 3. Security

Application workloads should operate within controlled security boundaries while reducing unnecessary exposure to external threats.

### Requirements

* Restrict direct exposure of application workloads.
* Enforce workload isolation.
* Implement namespace-level governance.
* Follow least-privilege principles.

### Architectural Impact

* Private worker nodes.
* Namespace security policies.
* Network segmentation.
* IAM role separation.

---

# 4. Reliability

The platform should provide predictable behavior during deployments, upgrades, and infrastructure changes.

### Requirements

* Prevent avoidable service interruptions.
* Support safe workload updates.
* Protect critical workloads during maintenance activities.

### Architectural Impact

* Pod Disruption Budgets.
* Controlled update strategies.
* Kubernetes health checks.
* Managed service adoption.

---

# 5. Resource Governance

The platform should prevent individual workloads from negatively impacting overall cluster stability.

### Requirements

* Control resource consumption.
* Establish workload boundaries.
* Ensure fair resource allocation.

### Architectural Impact

* Resource Quotas.
* Limit Ranges.
* Namespace isolation.
* Workload-level resource controls.

---

# 6. Cost Efficiency

Infrastructure resources should be utilized efficiently while maintaining platform performance and reliability.

### Requirements

* Avoid over-provisioning.
* Continuously optimize infrastructure utilization.
* Reduce unnecessary cloud spending.

### Architectural Impact

* Managed Kubernetes services.
* CAST AI optimization.
* Dynamic node sizing.
* Efficient workload placement.

---

# 7. Maintainability

The platform should be easy to operate, manage, and evolve over time.

### Requirements

* Standardized infrastructure deployment.
* Consistent environment management.
* Simplified operational workflows.

### Architectural Impact

* Infrastructure as Code.
* Modular Terraform design.
* Environment standardization.
* Kubernetes-native resource management.

---

# 8. Observability Readiness

The platform should expose sufficient operational data to support monitoring, troubleshooting, and future observability initiatives.

### Requirements

* Visibility into infrastructure health.
* Visibility into application behavior.
* Support operational troubleshooting.

### Architectural Impact

* Cluster logging.
* Metrics collection.
* Monitoring integration points.
* Platform telemetry capabilities.

---

# 9. Extensibility

The platform should serve as a foundation for future platform capabilities without requiring major architectural redesign.

### Requirements

* Support GitOps adoption.
* Support observability platforms.
* Support secrets management solutions.
* Support future platform services.

### Architectural Impact

* Kubernetes-based architecture.
* Modular infrastructure design.
* Platform-oriented resource organization.
* Standardized deployment model.

---

# Requirement Summary

| Requirement             | Objective                                  |
| ----------------------- | ------------------------------------------ |
| Availability            | Ensure platform continuity during failures |
| Scalability             | Support changing workload demand           |
| Security                | Protect workloads and infrastructure       |
| Reliability             | Minimize operational disruptions           |
| Resource Governance     | Prevent uncontrolled resource consumption  |
| Cost Efficiency         | Optimize infrastructure utilization        |
| Maintainability         | Simplify operations and platform evolution |
| Observability Readiness | Enable monitoring and troubleshooting      |
| Extensibility           | Support future platform capabilities       |

These non-functional requirements became the foundation for the architectural decisions, platform guardrails, and operational controls implemented throughout the AWS EKS platform.

