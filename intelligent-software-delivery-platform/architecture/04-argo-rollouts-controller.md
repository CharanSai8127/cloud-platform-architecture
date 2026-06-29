# Argo Rollouts Architecture

## Overview

Progressive Delivery extends traditional software deployment by continuously validating a release before exposing it to additional production users. While deployment strategies such as Canary Deployment define **how traffic should be introduced**, an additional control plane is required to manage the complete lifecycle of the rollout.

Kubernetes provides the native **Deployment** controller, which manages the lifecycle of application Pods. It ensures that the desired number of replicas are running, replaces failed Pods, and performs Rolling Updates. However, the Deployment controller considers its responsibility complete once the Pods become operational.

Enterprise software delivery requires significantly more than Pod lifecycle management. Modern deployment platforms must continuously evaluate production metrics, pause deployments, progressively expose users to new releases, and automatically recover when application health degrades.

This project introduces **Argo Rollouts**, a Kubernetes-native Progressive Delivery controller that extends Kubernetes with intelligent deployment capabilities through the use of Custom Resource Definitions (CRDs), controllers, reconciliation, traffic management, and production-aware decision making.

---

# Why Kubernetes Deployment Is Not Enough

The Kubernetes Deployment resource was designed to manage application availability rather than release quality.

A Deployment performs the following responsibilities:

* Creates ReplicaSets.
* Maintains Desired Replica Count.
* Performs Rolling Updates.
* Replaces Failed Pods.
* Validates Readiness and Liveness Probes.

Once every Pod becomes Ready, Kubernetes considers the deployment successful.

However, this assumption introduces an important limitation.

A Pod being Ready does **not** guarantee that:

* The application is serving requests correctly.
* The application performs within acceptable latency.
* Error rates remain within operational limits.
* The release is safe for all production users.

The Deployment controller therefore manages infrastructure health, but it cannot validate production behavior.

---

# Extending Kubernetes

One of Kubernetes' greatest strengths is its extensibility.

The Kubernetes API is not limited to native resources such as:

* Pods
* Deployments
* Services
* StatefulSets

It allows platform engineers to introduce entirely new resources by defining **Custom Resource Definitions (CRDs)**.

A CRD extends the Kubernetes API by introducing a new resource type together with its schema.

Once the resource exists, a controller continuously reconciles its desired state.

This extension model enables Kubernetes to manage almost any operational workflow.

Argo Rollouts leverages this capability by introducing the **Rollout** resource.

---

# Argo Rollouts

Argo Rollouts is a Progressive Delivery controller built specifically for Kubernetes.

Rather than replacing Kubernetes Deployments, Argo Rollouts extends Kubernetes by introducing additional deployment intelligence.

Its primary responsibilities include:

* Canary Deployments
* Blue-Green Deployments
* Progressive Traffic Shifting
* Automated Pauses
* Metric-Based Analysis
* Automated Promotion
* Automated Rollback

Instead of assuming that a running application represents a successful release, Argo Rollouts continuously validates production behavior before allowing the deployment to progress.

---

# Rollout Resource

The central resource introduced by Argo Rollouts is the **Rollout**.

The Rollout replaces the Kubernetes Deployment resource and becomes the desired state for the application's release lifecycle.

Unlike a Deployment, a Rollout describes much more than application replicas.

It defines:

* Deployment Strategy
* Traffic Weights
* Rollout Steps
* Pause Durations
* Analysis Stages
* Promotion Rules
* Rollback Conditions

The Rollout therefore represents the desired behavior of the software release rather than simply the desired number of application Pods.

---

# Rollout Controller

The Rollout resource alone performs no work.

Like every Kubernetes resource, it requires a controller.

The **Argo Rollouts Controller** continuously watches every Rollout resource and reconciles its current state toward the desired state.

Whenever a new application version is deployed, the controller becomes responsible for managing the complete rollout process.

Its responsibilities include:

* Creating ReplicaSets.
* Scaling Stable and Canary versions.
* Managing rollout steps.
* Executing pauses.
* Updating traffic weights.
* Running AnalysisTemplates.
* Querying Prometheus.
* Promoting healthy releases.
* Rolling back failed releases.

The controller therefore becomes the decision engine of the deployment platform.

---

# Reconciliation

Like every Kubernetes controller, Argo Rollouts follows the reconciliation model.

Rather than executing a deployment once and terminating, the controller continuously compares:

```text
Desired State

↓

Current State

↓

Difference?

↓

Reconcile
```

If differences exist, the controller performs the required actions to move the rollout toward the desired state.

This continuous reconciliation enables the rollout to react automatically to changing production conditions.

---

# Stable And Canary ReplicaSets

During a Canary deployment, two ReplicaSets coexist.

```text
Stable ReplicaSet

(Current Production)
```

```text
Canary ReplicaSet

(New Version)
```

Initially, nearly all production traffic continues reaching the Stable ReplicaSet.

Only a small percentage reaches the Canary ReplicaSet.

As the rollout progresses, traffic gradually shifts toward the Canary version.

The Stable ReplicaSet remains available throughout the rollout, allowing immediate recovery whenever necessary.

---

# Rollout Lifecycle

The complete rollout lifecycle consists of several stages.

```text
New Version

↓

Create Canary ReplicaSet

↓

Shift Initial Traffic

↓

Pause

↓

Execute Analysis

↓

Application Healthy?
```

If validation succeeds:

```text
Increase Traffic

↓

Repeat Analysis

↓

100% Traffic

↓

Promote Stable Version
```

If validation fails:

```text
Abort Rollout

↓

Redirect Traffic

↓

Stable Version Continues

↓

Canary Removed
```

Unlike Rolling Updates, deployment progression depends upon application behavior rather than Pod readiness alone.

---

# Rollout Steps

A Rollout progresses through a predefined sequence of rollout steps.

Typical steps include:

* Shift Traffic
* Pause
* Execute Analysis
* Continue Rollout

For example:

```text
5%

↓

Pause

↓

20%

↓

Pause

↓

50%

↓

Pause

↓

100%
```

Each step provides an opportunity to validate production behavior before exposing additional users.

---

# Automated Pause

One of the key capabilities of Argo Rollouts is the ability to automatically pause deployment progression.

Unlike traditional deployments that continuously replace Pods until completion, Rollouts intentionally stop after each traffic increment.

The pause allows the controller to:

* Observe Production Traffic.
* Collect Metrics.
* Execute Analysis.
* Determine Application Health.

Deployment therefore progresses only after sufficient operational evidence has been collected.

---

# Analysis

Every pause is typically followed by an analysis stage.

The controller executes an AnalysisTemplate that queries an external metrics provider such as Prometheus.

The collected metrics are compared against predefined success criteria.

Examples include:

* HTTP Success Rate
* Error Rate
* Request Latency
* Availability

If the metrics satisfy the configured thresholds, the rollout proceeds.

Otherwise, corrective action is initiated.

---

# Promotion

Promotion represents the successful completion of the rollout.

Promotion occurs only after:

* Every rollout step succeeds.
* Every analysis succeeds.
* The application remains healthy.
* 100% of production traffic reaches the new version.

Only then does the Canary ReplicaSet become the new Stable ReplicaSet.

The previous Stable ReplicaSet is gradually removed.

---

# Automated Rollback

If production metrics indicate application degradation, Argo Rollouts automatically aborts the rollout.

Traffic immediately returns to the Stable ReplicaSet while the Canary version is isolated from production traffic.

The rollback process requires no manual intervention.

Instead, the controller continuously enforces the deployment policy defined by the Rollout resource.

This significantly reduces recovery time while minimizing customer impact.

---

# Complete Rollout Architecture

```text
Git Repository
        │
        ▼
ArgoCD
        │
        ▼
Rollout Resource
        │
        ▼
Argo Rollouts Controller
        │
        ▼
Stable ReplicaSet
        │
        │
Canary ReplicaSet
        │
        ▼
Traffic Management
        │
        ▼
Pause
        │
        ▼
AnalysisTemplate
        │
        ▼
Prometheus
        │
        ▼
Healthy?
   │            │
   ▼            ▼
Promote     Rollback
```

Every component contributes to a closed-loop deployment process where production feedback continuously influences deployment progression.

---

# Kubernetes Deployment vs Argo Rollouts

| Kubernetes Deployment               | Argo Rollouts                               |
| ----------------------------------- | ------------------------------------------- |
| Native Kubernetes Controller        | Progressive Delivery Controller             |
| Rolling Updates                     | Canary & Blue-Green                         |
| Manages Pods                        | Manages Software Releases                   |
| Readiness & Liveness Checks         | Production Metric Validation                |
| Deployment Ends When Pods Are Ready | Rollout Ends When Release Is Proven Healthy |
| Manual Promotion                    | Automated Promotion                         |
| Manual Rollback                     | Automated Rollback                          |
| No Traffic Management               | Progressive Traffic Management              |

The Deployment controller ensures that the infrastructure is operational.

The Rollouts controller ensures that the software release is operational.

These are fundamentally different responsibilities.

---

# Advantages

Adopting Argo Rollouts introduces several enterprise capabilities.

These include:

* Progressive Delivery.
* Intelligent Traffic Management.
* Continuous Reconciliation.
* Automated Analysis.
* Production Validation.
* Automated Promotion.
* Automated Rollback.
* Reduced Deployment Risk.
* Faster Recovery.
* Improved Production Reliability.

Rather than treating deployment as a one-time operation, Argo Rollouts transforms deployment into a continuously managed process.

---

# Key Takeaways

Argo Rollouts extends Kubernetes by introducing a dedicated controller responsible for managing the lifecycle of software releases rather than simply managing application Pods.

By leveraging Kubernetes' extensibility through Custom Resource Definitions and controllers, Argo Rollouts replaces the traditional Deployment resource with the Rollout resource, enabling Progressive Delivery through controlled traffic shifting, automated pauses, production-aware analysis, promotion, and rollback.

The result is a deployment platform where release decisions are no longer based solely on infrastructure readiness, but on continuous evaluation of the application's behavior under real production traffic. This shifts software delivery from a manual, time-driven activity to an intelligent, feedback-driven operational process capable of delivering reliable production releases.

