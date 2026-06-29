# Progressive Delivery

## Overview

Modern software delivery extends far beyond successfully deploying an application into production. While traditional deployment strategies focus on replacing one application version with another, they often assume that a deployment is successful once the application Pods become operational. In reality, a running application is not necessarily a healthy application.

Production environments continuously experience changing workloads, varying traffic patterns, infrastructure constraints, and unpredictable user behavior. These conditions make deployment one of the highest-risk operations within the software delivery lifecycle.

Canary Deployment significantly reduces this risk by gradually exposing users to the new application version. However, Canary itself only controls **how traffic is introduced** to the application. It does not determine whether the deployment should continue, pause, or be rolled back.

This limitation gave rise to **Progressive Delivery**, an operational methodology that combines controlled traffic management with continuous production validation. Instead of assuming a deployment is successful, Progressive Delivery continuously observes the running application, evaluates production metrics, and automatically decides whether the rollout should continue or be aborted.

---

# From Deployment To Release Validation

Traditional deployment pipelines typically follow a straightforward process.

```text
Build

↓

Test

↓

Deploy

↓

Application Running
```

Once the application becomes operational, engineers manually monitor dashboards and production metrics to determine whether the release is behaving correctly.

This process introduces several operational challenges.

* Manual Monitoring
* Delayed Failure Detection
* Human Decision Making
* Slower Recovery
* Larger Blast Radius

Modern cloud-native platforms therefore extend deployment beyond simply starting application containers.

The release itself must continuously prove that it is healthy before receiving additional production traffic.

---

# What Is Progressive Delivery?

Progressive Delivery is a software delivery methodology that gradually releases new application versions while continuously evaluating their behavior in production before exposing them to additional users.

Rather than treating deployment as a single event, Progressive Delivery transforms deployment into a continuous feedback-driven process.

The objective is no longer simply to deploy software.

The objective becomes:

* Release Software Safely.
* Observe Production Behavior.
* Validate Application Health.
* Promote Healthy Releases.
* Automatically Recover From Failures.

Deployment decisions are therefore based on production feedback instead of elapsed time.

---

# Why Canary Deployment Alone Is Not Enough

Canary Deployment significantly improves deployment safety by introducing traffic gradually.

For example:

```text
5%

↓

20%

↓

50%

↓

100%
```

Only a limited number of users initially receive the new application version.

If failures occur, fewer users are affected.

However, Canary Deployment only answers one question.

> **How should traffic be shifted?**

It does not answer:

* Is the application healthy?
* Should traffic continue increasing?
* Should deployment pause?
* Should the rollout be promoted?
* Should rollback occur?

Without additional intelligence, these decisions remain manual.

Progressive Delivery extends Canary Deployment by automating these operational decisions.

---

# Progressive Delivery Controller

A Progressive Delivery Controller is responsible for managing the complete lifecycle of a software release.

Instead of deploying the application and immediately completing the deployment, the controller continuously observes the running application before deciding whether additional production traffic should be introduced.

Its responsibilities include:

* Progressive Traffic Shifting
* Automated Pauses
* Production Observation
* Metric Collection
* Health Evaluation
* Automated Promotion
* Automated Rollback

Unlike Kubernetes' native Deployment controller, the Progressive Delivery controller continuously evaluates the health of the software release rather than simply managing application Pods.

---

# Rollout Resource

Kubernetes provides the Deployment resource for application deployment.

This project replaces the Deployment resource with the **Rollout** custom resource provided by Argo Rollouts.

The Rollout resource extends Kubernetes by introducing deployment capabilities that are not available through native Deployments.

A Rollout defines:

* Deployment Strategy
* Traffic Shift Percentages
* Pause Durations
* Analysis Stages
* Promotion Conditions
* Rollback Behavior

Rather than completing deployment once Pods become Ready, the Rollout continues until the release itself has successfully passed every validation stage.

---

# AnalysisTemplate

Traffic progression should not depend solely on time.

Instead, every deployment stage should validate the health of the running application.

This responsibility belongs to the **AnalysisTemplate**.

The AnalysisTemplate defines:

* Which metrics should be evaluated.
* Which monitoring system should be queried.
* Success Conditions.
* Failure Conditions.
* Evaluation Intervals.
* Number of Validation Attempts.

During every rollout stage, the Progressive Delivery controller executes the AnalysisTemplate and determines whether the deployment should continue.

---

# Metric-Based Decision Making

Traditional deployments rely primarily on infrastructure health.

Examples include:

* Pod Running
* Readiness Probe
* Liveness Probe

These checks verify that containers are operational.

They do not determine whether the application is behaving correctly under production traffic.

Progressive Delivery extends this validation by evaluating operational metrics such as:

* HTTP Success Rate
* HTTP Error Rate
* Request Latency
* Availability
* CPU Utilization
* Memory Utilization

The rollout therefore validates both infrastructure health and application behavior before continuing.

---

# Progressive Rollout Lifecycle

The rollout follows a continuous validation process.

```text
Deploy New Version

↓

Shift Traffic

↓

Pause

↓

Collect Metrics

↓

Analyze Results

↓

Healthy?
```

If the application satisfies every validation rule:

```text
Continue

↓

Increase Traffic

↓

Repeat Validation
```

If validation fails:

```text
Abort

↓

Rollback

↓

Stable Version Continues Serving Traffic
```

The previous stable release remains available throughout the rollout process, ensuring rapid recovery whenever failures occur.

---

# Kubernetes Deployment vs Rollout

Both Deployments and Rollouts manage application releases, but they operate at different levels of responsibility.

| Kubernetes Deployment                    | Rollout                                          |
| ---------------------------------------- | ------------------------------------------------ |
| Native Kubernetes Resource               | Custom Resource (CRD)                            |
| Rolling Updates                          | Canary & Blue-Green Strategies                   |
| Pod Lifecycle Management                 | Software Release Lifecycle                       |
| Readiness & Liveness Validation          | Production Metric Validation                     |
| Manual Promotion Decisions               | Automated Promotion                              |
| Manual Rollback                          | Automated Rollback                               |
| Deployment Completes When Pods Are Ready | Rollout Completes When Release Is Proven Healthy |

A Deployment determines whether application Pods are operational.

A Rollout determines whether the software release is operational.

This distinction forms the foundation of Progressive Delivery.

---

# Progressive Delivery Architecture

The deployment process becomes a closed feedback loop.

```text
New Application Version
           │
           ▼
     Rollout Resource
           │
           ▼
Progressive Delivery Controller
           │
           ▼
     Shift Traffic
           │
           ▼
 Pause & Observe
           │
           ▼
 Execute Analysis
           │
           ▼
Collect Production Metrics
           │
           ▼
Application Healthy?
     │               │
     ▼               ▼
Promote         Rollback
```

Rather than assuming success, every rollout stage continuously validates the application before exposing additional users.

---

# Advantages Of Progressive Delivery

Progressive Delivery introduces several operational improvements.

These include:

* Reduced Deployment Risk
* Progressive User Exposure
* Continuous Production Validation
* Automated Decision Making
* Faster Failure Detection
* Smaller Blast Radius
* Improved Reliability
* Reduced Human Error
* Safe Production Releases

By shifting deployment decisions from manual operators to Kubernetes controllers, software delivery becomes significantly more reliable and predictable.

---

# From Time-Based To Feedback-Driven Deployments

Traditional deployment strategies progress according to predefined timing.

```text
Deploy

↓

Wait

↓

Continue
```

Progressive Delivery fundamentally changes this model.

Deployment progression now depends upon production feedback.

```text
Deploy

↓

Observe

↓

Measure

↓

Evaluate

↓

Continue

or

Rollback
```

This transformation represents one of the most significant advancements in modern software delivery.

Instead of deploying software based solely on infrastructure readiness, deployments are continuously validated using real production behavior.

---

# Key Takeaways

Progressive Delivery extends traditional deployment strategies by introducing continuous production validation into the software delivery process.

While Canary Deployment provides the ability to gradually expose users to a new application version, Progressive Delivery introduces the intelligence required to determine whether that exposure should continue.

By replacing Kubernetes Deployments with Rollouts, defining validation policies through AnalysisTemplates, and evaluating production metrics before every rollout stage, the platform transforms deployment from a manual, time-driven process into an automated, feedback-driven system.

This architecture enables safer software releases, reduces operational risk, minimizes deployment blast radius, and establishes a reliable foundation for modern cloud-native application delivery.

