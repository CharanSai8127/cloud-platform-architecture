# Deployment Strategies

## Overview

Every software deployment introduces uncertainty into a production environment. Regardless of how thoroughly an application has been tested, production always contains variables that cannot be completely reproduced during development or staging. As applications continue to evolve, organizations require mechanisms that minimize deployment risk while maintaining application availability and ensuring a positive user experience.

Deployment strategies define **how a new version of an application replaces an existing version**. Each strategy represents a different balance between deployment speed, operational complexity, risk, rollback capability, and infrastructure requirements.

Over time, deployment strategies have evolved from simple replacement approaches to intelligent deployment models capable of validating software while it is being released.

---

# Why Deployment Strategies Exist

The primary objective of a deployment strategy is not simply to deploy software.

Its objective is to answer several important questions:

* How should a new version be introduced?
* How much production traffic should reach the new version?
* How can downtime be avoided?
* How can deployment risk be minimized?
* How quickly can failures be recovered?
* How much of the existing infrastructure should be preserved?

Different deployment strategies answer these questions differently depending on the operational requirements of the application.

---

# Recreate Deployment

The Recreate strategy is the simplest deployment model.

The existing application version is completely terminated before the new version is deployed.

```text
Version 1

↓

Terminate

↓

Deploy Version 2
```

Since the previous version no longer exists during deployment, users experience service interruption until the new version becomes available.

### Advantages

* Simple implementation.
* Minimal infrastructure requirements.
* Easy to understand.

### Limitations

* Service Downtime.
* High Deployment Risk.
* No Immediate Rollback.
* Unsuitable for production systems requiring high availability.

Although suitable for development or non-critical environments, the Recreate strategy is rarely adopted for enterprise production workloads.

---

# Rolling Update

To eliminate downtime, Kubernetes introduced the Rolling Update deployment strategy through its native Deployment resource.

Instead of replacing the application all at once, Kubernetes gradually replaces older Pods with newer Pods while maintaining application availability.

```text
Version 1

↓

25%

↓

50%

↓

75%

↓

100%
```

At every stage, Kubernetes ensures that a minimum number of Pods remain available before terminating older instances.

This allows applications to remain online throughout the deployment process.

### Advantages

* No Downtime.
* Native Kubernetes Support.
* Automated Pod Replacement.
* Efficient Resource Utilization.

### Limitations

The Kubernetes Deployment controller focuses exclusively on Pod lifecycle management.

Its responsibility ends once:

* Pods are Running.
* Readiness Probes succeed.
* Desired Replica Count is achieved.

It does not evaluate:

* Application Error Rate.
* Request Latency.
* Business Metrics.
* User Experience.
* Production Health.

The Deployment controller therefore assumes that a Ready Pod represents a successful deployment.

In reality, a Ready Pod does not necessarily indicate a healthy software release.

---

# Blue-Green Deployment

Blue-Green Deployment introduces two independent application environments.

```text
Blue Environment

(Current Production)

Green Environment

(New Release)
```

Only one environment receives production traffic.

The second environment remains isolated until deployment is complete.

Once validation succeeds, traffic switches entirely from the Blue environment to the Green environment.

```text
Blue

↓

Switch

↓

Green
```

If the deployment fails, traffic immediately returns to the previous environment.

### Advantages

* Zero Downtime.
* Immediate Rollback.
* Environment Isolation.
* Reduced Upgrade Risk.

### Limitations

* Duplicate Infrastructure.
* Higher Resource Cost.
* Binary Traffic Switch.
* No Gradual User Exposure.

Although Blue-Green significantly improves deployment safety, every production user is still exposed to the new version immediately after traffic switching.

---

# Canary Deployment

Canary Deployment further reduces deployment risk by introducing the new version gradually.

Instead of switching every user simultaneously, only a small percentage of traffic reaches the new version initially.

```text
5%

↓

20%

↓

50%

↓

100%
```

The remaining traffic continues using the stable application version.

As confidence increases, additional users are gradually exposed to the new release.

If unexpected behavior appears, only a limited portion of users is affected.

This significantly reduces the deployment blast radius.

### Advantages

* Reduced Production Risk.
* Progressive User Exposure.
* Minimal Customer Impact.
* Safer Production Releases.

### Limitations

Traditional Canary Deployment defines **how traffic should be shifted**, but it does not define **how deployment decisions should be made**.

Questions such as:

* Should traffic continue increasing?
* Should deployment pause?
* Should rollback occur?

are typically answered through manual observation.

Canary therefore improves traffic management but does not fully automate release management.

---

# A/B Deployment

Unlike Canary Deployment, A/B Deployment is not primarily intended to reduce deployment risk.

Instead, it is used to compare multiple application variations simultaneously.

Different groups of users receive different versions of the application.

```text
Users

├── Version A

└── Version B
```

Application behavior is then evaluated using business metrics such as:

* User Engagement.
* Click-Through Rate.
* Conversion Rate.
* Feature Adoption.

The objective is product experimentation rather than deployment safety.

---

# Feature Flags

Feature Flags separate **software deployment** from **feature release**.

New functionality is deployed together with the existing application but remains disabled until explicitly activated.

```text
Deploy

↓

Feature Disabled

↓

Enable Feature

↓

Users Access Feature
```

Feature Flags provide:

* Gradual Feature Release.
* User Segmentation.
* Fast Feature Disable.
* Controlled Experimentation.

Although Feature Flags improve release flexibility, they do not replace deployment strategies.

Instead, they complement them.

---

# Comparing Deployment Strategies

| Strategy       | Downtime | Rollback  | Traffic Control      | Production Risk                |
| -------------- | -------- | --------- | -------------------- | ------------------------------ |
| Recreate       | High     | Slow      | None                 | Very High                      |
| Rolling Update | None     | Manual    | Pod Replacement Only | Medium                         |
| Blue-Green     | None     | Immediate | Binary Switch        | Low                            |
| Canary         | None     | Fast      | Progressive          | Very Low                       |
| Feature Flags  | None     | Immediate | Feature-Level        | Depends on Deployment Strategy |

Each strategy represents an improvement in deployment safety, operational flexibility, and user impact.

---

# Why This Project Uses Canary Deployment

This project adopts the Canary deployment strategy because it minimizes deployment risk while maintaining continuous application availability.

Rather than exposing every user to the new release immediately, the application is introduced gradually through controlled traffic percentages.

This provides several operational advantages.

* Reduced Blast Radius.
* Safer Production Validation.
* Gradual User Exposure.
* Continuous Health Evaluation.
* Faster Failure Detection.

However, Canary Deployment alone is only responsible for **progressively shifting traffic**.

It does not determine whether the deployment should continue, pause, or roll back.

Those decisions still require additional intelligence.

---

# From Canary To Progressive Delivery

Canary Deployment answers the question:

> **How should traffic be introduced to a new application version?**

It does **not** answer:

* Is the application healthy?
* Should deployment continue?
* Should traffic increase?
* Should the rollout pause?
* Should the release be rolled back?

These decisions require continuous analysis of the running application.

This introduces the concept of **Progressive Delivery**, where deployment decisions are driven by production feedback rather than elapsed time or manual observation.

Progressive Delivery extends Canary Deployment by combining:

* Controlled Traffic Shifting.
* Continuous Observation.
* Metric-Based Validation.
* Automated Promotion.
* Automated Rollback.

Instead of simply deploying software, the platform continuously validates whether the release is safe before exposing it to additional users.

---

# Key Takeaways

Deployment strategies have evolved alongside modern software systems to reduce production risk while maintaining application availability.

Beginning with simple replacement deployments, the industry gradually introduced Rolling Updates, Blue-Green Deployments, and Canary Deployments to improve deployment safety.

Among these strategies, Canary Deployment provides the foundation for gradually exposing new software to production users. However, traffic management alone is insufficient for modern cloud-native platforms.

Enterprise platforms require deployment decisions to be driven by production feedback rather than manual observation.

This evolution leads directly to **Progressive Delivery**, where software releases are continuously validated using operational metrics before being fully promoted into production.

