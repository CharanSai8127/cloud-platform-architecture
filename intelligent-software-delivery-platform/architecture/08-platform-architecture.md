# Platform Architecture

## Overview

Building a reliable software delivery platform requires much more than deploying an application into Kubernetes. Modern cloud-native platforms combine infrastructure provisioning, networking, container orchestration, traffic management, observability, and deployment automation into a unified architecture capable of delivering software safely and continuously.

This project demonstrates a complete Azure-based Progressive Delivery platform where every architectural layer performs a well-defined responsibility while collaborating to automate software deployment. Rather than relying on manual operational activities, the platform continuously manages infrastructure, application deployment, traffic distribution, health validation, and rollback through Kubernetes controllers.

The architecture follows a layered approach, where each layer builds upon the capabilities of the previous one to create an intelligent software delivery platform.

---

# Architectural Principles

The platform is designed around several core engineering principles.

* Infrastructure as Code
* Separation of Responsibilities
* Declarative Infrastructure
* Kubernetes Native Operations
* Progressive Delivery
* Feedback-Driven Decisions
* GitOps
* Automation
* Reliability
* High Availability

Each architectural decision contributes towards reducing deployment risk while improving operational consistency and production reliability.

---

# Layered Architecture

The complete platform is organized into multiple architectural layers.

```text
                    Users
                       │
                       ▼
                Gateway API
                 (HTTPRoute)
                       │
                       ▼
            Traffic Management Layer
                       │
                       ▼
          Progressive Delivery Layer
              (Argo Rollouts)
                       │
          ┌────────────┴────────────┐
          ▼                         ▼
     Stable Version          Canary Version
                       │
                       ▼
             Kubernetes Platform
                 (Azure AKS)
                       │
                       ▼
             Observability Layer
          (Prometheus + Analysis)
                       │
                       ▼
          Deployment Decision Engine
                       │
                       ▼
         Azure Infrastructure Layer
                  (Terraform)
```

Every layer has a dedicated responsibility and communicates with adjacent layers through well-defined Kubernetes resources.

---

# Infrastructure Layer

The foundation of the platform is provisioned using Terraform.

Terraform provides Infrastructure as Code (IaC), allowing Azure resources to be created declaratively, version controlled, and consistently reproduced across environments.

The infrastructure layer provisions:

* Resource Group
* Virtual Network
* Public Subnet
* Private Subnet
* Network Security Groups
* NAT Gateway
* Public IP
* Azure Kubernetes Service (AKS)

Rather than manually provisioning infrastructure, every component is described declaratively, allowing Terraform to reconcile Azure resources to the desired state.

---

# Networking Architecture

The networking layer separates infrastructure resources from application workloads by introducing public and private subnets.

The **Public Subnet** hosts networking resources responsible for external connectivity.

These include:

* NAT Gateway
* Public IP Resources
* Azure Load Balancer
* Network Interfaces

The **Private Subnet** hosts the Kubernetes platform.

This includes:

* AKS Nodes
* Kubernetes Pods
* Platform Components
* Application Workloads

Applications never receive public IP addresses.

Using Azure CNI, every Pod receives a private IP address from the subnet associated with the AKS node pool. Outbound communication is translated through the NAT Gateway, while inbound communication remains protected behind the Gateway API.

This architecture significantly reduces the attack surface by ensuring that only networking infrastructure is exposed externally.

---

# Kubernetes Platform

Azure Kubernetes Service (AKS) provides the runtime responsible for executing containerized workloads.

The platform hosts:

* Application Pods
* Services
* Horizontal Pod Autoscaler
* Gateway API
* Argo Rollouts
* Prometheus
* Supporting Kubernetes Controllers

Rather than simply running containers, Kubernetes continuously reconciles the desired application state by scheduling Pods, maintaining ReplicaSets, replacing failed containers, and exposing workloads through Services.

---

# Traffic Management Layer

Traffic enters the platform through the Gateway API.

The Gateway provides the entry point into the Kubernetes cluster, while HTTPRoute defines how requests are distributed across application versions.

During Progressive Delivery, HTTPRoute dynamically adjusts traffic weights between:

* Stable Service
* Canary Service

Rather than exposing every user to the newest release immediately, traffic is introduced gradually according to the rollout strategy defined by Argo Rollouts.

This separation allows networking infrastructure and application routing to evolve independently.

---

# Progressive Delivery Layer

The Progressive Delivery layer is responsible for managing the lifecycle of software releases.

Instead of using Kubernetes Deployments, the platform adopts the Rollout custom resource provided by Argo Rollouts.

The Rollout defines:

* Deployment Strategy
* Traffic Percentages
* Pause Stages
* Analysis Stages
* Promotion Rules
* Rollback Conditions

The Argo Rollouts controller continuously reconciles the rollout, progressively exposing production traffic while validating application health before allowing additional users to access the new version.

---

# Observability Layer

Deployment decisions require continuous visibility into the behaviour of the running application.

This responsibility belongs to the Observability layer.

Prometheus continuously collects metrics exposed by the application through ServiceMonitor resources.

Typical metrics include:

* Request Rate
* Success Rate
* Error Rate
* Request Latency
* CPU Utilization
* Memory Utilization

Rather than serving only operational dashboards, these metrics become direct inputs to the Progressive Delivery process.

---

# Analysis Layer

AnalysisTemplates define the validation policy used during every rollout stage.

Each AnalysisTemplate specifies:

* Data Source
* Prometheus Query
* Success Criteria
* Failure Criteria
* Evaluation Interval
* Retry Behaviour

At every rollout step, the Progressive Delivery controller executes the AnalysisTemplate and determines whether the deployment should continue.

Deployment progression therefore depends upon measurable application behaviour rather than elapsed time.

---

# Deployment Decision Engine

The Argo Rollouts controller acts as the platform's deployment decision engine.

Its responsibilities include:

* Reading Rollout Resources
* Updating HTTPRoute Traffic Weights
* Executing AnalysisTemplates
* Querying Prometheus
* Evaluating Metrics
* Promoting Healthy Releases
* Rolling Back Failed Releases

Unlike traditional deployment systems, deployment decisions are continuously evaluated throughout the rollout lifecycle.

---

# GitOps Layer

Application deployment is managed declaratively using ArgoCD.

Rather than applying Kubernetes manifests manually, the desired cluster state is stored within Git repositories.

The deployment flow becomes:

```text
Developer

↓

Git Repository

↓

ArgoCD

↓

Rollout Resource

↓

Argo Rollouts

↓

Gateway API

↓

Prometheus

↓

Deployment Decision
```

Git therefore becomes the single source of truth for both infrastructure configuration and application deployment.

---

# Complete Platform Flow

The interaction between platform components follows a continuous feedback loop.

```text
Developer

        │

        ▼

Git Repository

        │

        ▼

ArgoCD Synchronization

        │

        ▼

Rollout Resource

        │

        ▼

Argo Rollouts Controller

        │

        ▼

HTTPRoute Updates

        │

        ▼

Traffic Split

        │

        ▼

Application Pods

        │

        ▼

Prometheus Metrics

        │

        ▼

AnalysisTemplate

        │

        ▼

Deployment Decision

        │

 ┌──────┴────────┐

 ▼               ▼

Promote      Rollback
```

Rather than treating deployment as a single event, the platform continuously observes and validates every rollout stage before progressing further.

---

# Separation Of Responsibilities

One of the most important architectural principles implemented within this platform is clear separation of responsibilities.

| Component        | Responsibility                   |
| ---------------- | -------------------------------- |
| Terraform        | Infrastructure Provisioning      |
| Azure Networking | Network Isolation & Connectivity |
| AKS              | Container Orchestration          |
| Gateway API      | Traffic Management               |
| HTTPRoute        | Weighted Request Routing         |
| ArgoCD           | GitOps Reconciliation            |
| Argo Rollouts    | Progressive Delivery             |
| Prometheus       | Metrics Collection               |
| AnalysisTemplate | Deployment Validation            |
| HPA              | Dynamic Scaling                  |

Every component performs a single responsibility while collaborating with other controllers to deliver reliable software deployments.

---

# Architectural Advantages

This layered architecture provides several operational advantages.

These include:

* Declarative Infrastructure
* Kubernetes Native Operations
* Secure Network Segmentation
* Progressive Traffic Management
* Automated Deployment Decisions
* Continuous Production Validation
* GitOps-Based Configuration Management
* Automated Promotion
* Automated Rollback
* Reduced Operational Risk
* Improved Reliability
* High Availability

Each layer contributes independently while collectively creating a resilient software delivery platform.

---

# Key Takeaways

The platform architecture demonstrates how modern cloud-native systems combine infrastructure, Kubernetes, networking, observability, and automation into a single deployment platform capable of delivering software safely and continuously.

Terraform provisions the Azure infrastructure, AKS provides the container orchestration platform, the Gateway API manages traffic, Argo Rollouts controls Progressive Delivery, Prometheus supplies production telemetry, and AnalysisTemplates validate application health. These components are integrated through GitOps using ArgoCD, enabling every deployment to be reconciled, observed, validated, promoted, or rolled back automatically.

Rather than functioning as isolated technologies, each layer contributes a specific operational capability to the overall platform. Together, they transform software delivery from a manual deployment process into an intelligent, feedback-driven architecture that continuously reduces deployment risk while improving reliability and operational consistency.

