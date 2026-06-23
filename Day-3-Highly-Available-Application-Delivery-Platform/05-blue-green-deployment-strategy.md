# Blue-Green Deployment Strategy

## Overview

Applications continuously evolve over time.

Business requirements change, new features are introduced, bugs are fixed, dependencies are upgraded, and security vulnerabilities are addressed. As a result, newer versions of the application must be deployed into production regularly.

While deploying a new version appears simple, every release introduces risk.

Examples include:

- Application Startup Failures
- Runtime Errors
- Configuration Issues
- Dependency Conflicts
- Unexpected Platform Constraints

A failed deployment can impact application availability and user experience.

The objective of a deployment strategy is therefore not simply to deploy a new version, but to deploy it safely while minimizing risk and preserving availability.

---

# Why Deployment Strategies Exist

Application releases are one of the most common sources of operational incidents.

A deployment may introduce:

- Incorrect Business Logic
- Performance Issues
- Application Crashes
- Infrastructure Compatibility Problems

If the currently running version is directly replaced with a new version, recovery may require another deployment operation.

Example:

```text
Current Version
        ↓
Deploy New Version
        ↓
Failure
        ↓
Service Disruption
```

As deployment frequency increases, the probability of deployment-related incidents also increases.

To reduce this risk, deployment strategies are used to separate application releases from user-facing traffic.

---

# Common Deployment Strategies

Several deployment strategies exist to introduce application changes.

## Recreate Strategy

The existing version is terminated before the new version is deployed.

Example:

```text
Old Version
        ↓
Shutdown
        ↓
Deploy New Version
```

### Advantages

- Simple Implementation

### Challenges

- Downtime During Deployment
- No Immediate Rollback Path

---

## Rolling Update Strategy

Application replicas are replaced gradually.

Example:

```text
Version 1
        ↓
Version 1 + Version 2
        ↓
Version 2
```

### Advantages

- Reduced Downtime
- Native Kubernetes Support

### Challenges

- Multiple Versions Running Simultaneously
- Rollback Requires Additional Deployment Activity

---

## Canary Strategy

A small percentage of users receive the new version before broader rollout.

Example:

```text
95% → Current Version

5% → New Version
```

### Advantages

- Controlled Exposure
- Progressive Validation

### Challenges

- Increased Operational Complexity
- Traffic Weight Management Required

---

## Blue-Green Strategy

Two complete application environments exist simultaneously.

Example:

```text
Blue Environment
        ↓
Current Production Version

Green Environment
        ↓
New Version
```

Traffic is switched only after validation succeeds.

### Advantages

- Zero-Downtime Deployment
- Fast Rollback
- Clear Environment Separation

---

# Why Blue-Green Was Chosen

The primary objective of this platform is safe application delivery.

The deployment architecture must satisfy:

- Application Availability
- Deployment Safety
- Fast Recovery
- Operational Simplicity

Blue-Green deployment aligns well with these requirements.

Unlike direct replacement strategies, Blue-Green separates:

```text
Application Deployment

from

Traffic Exposure
```

This allows the new version to be deployed and validated independently before receiving production traffic.

---

# Blue Environment

The Blue environment represents the currently active production version.

### Characteristics

- Receives Production Traffic
- Serves Active Users
- Represents The Stable Version

Example:

```text
Users
        ↓
Gateway
        ↓
Blue Environment
```

At any given time, one environment acts as the active production environment.

---

# Green Environment

The Green environment represents the newly deployed application version.

### Characteristics

- Receives No Production Traffic Initially
- Used For Validation
- Represents The Candidate Release

Example:

```text
Green Environment
        ↓
New Application Version
```

The Green environment allows deployment activities to occur without impacting active users.

---

# Blue-Green Deployment Lifecycle

The deployment process follows a controlled sequence.

## Step 1: Blue Is Active

The stable application version serves all production traffic.

```text
Users
        ↓
Gateway
        ↓
Blue
```

---

## Step 2: Deploy Green

A new version is deployed into the Green environment.

```text
Blue
        ↓
Active

Green
        ↓
New Version
```

At this stage, users continue interacting with Blue.

---

## Step 3: Validate Green

The new version is verified before traffic is introduced.

Validation may include:

- Startup Verification
- Readiness Checks
- Health Checks
- Smoke Testing

The objective is to confirm that the application behaves as expected before becoming user-facing.

---

## Step 4: Switch Traffic

Once validation succeeds:

```text
Users
        ↓
Gateway
        ↓
Green
```

Traffic is redirected to the Green environment.

The Green version now becomes the active production version.

---

## Step 5: Blue Becomes Standby

The previous version remains available as a recovery option.

```text
Green
        ↓
Active

Blue
        ↓
Standby
```

This creates a rapid rollback path if issues are discovered after release.

---

# Validation Before Traffic Exposure

One of the key advantages of Blue-Green deployments is separating deployment from production traffic.

A deployment does not automatically become visible to users.

Instead:

```text
Deploy
        ↓
Validate
        ↓
Expose Traffic
```

This reduces deployment risk and improves release confidence.

Validation can include:

- Readiness Probes
- Liveness Probes
- Application Health Checks
- Functional Verification

Only after validation succeeds is production traffic redirected.

---

# Rollback Strategy

Failures are expected.

The deployment architecture must provide a simple recovery mechanism.

If the Green environment experiences issues:

```text
Users
        ↓
Gateway
        ↓
Blue
```

Traffic is redirected back to the previous version.

The rollback operation becomes:

```text
Traffic Change

not

Application Redeployment
```

This significantly reduces recovery time and operational effort.

---

# Shared Database Considerations

The Blue and Green environments represent different versions of the same application.

They are not independent systems.

Both environments consume the same MySQL InnoDB Cluster.

Example:

```text
Blue Application
        ↓
Shared Database

Green Application
        ↓
Shared Database
```

The database remains the authoritative source of business data.

### Benefits

- Single Source Of Truth
- Simplified Data Management
- Reduced Infrastructure Duplication
- Faster Environment Transitions

When using a shared database model, schema changes must remain compatible across both application versions during deployment transitions.

---

# Operational Benefits

The Blue-Green deployment model provides several operational advantages.

## Improved Availability

Users continue interacting with the active environment while new versions are deployed.

---

## Safer Releases

The new version can be validated before receiving production traffic.

---

## Faster Recovery

Rollback becomes a traffic-switching operation rather than a redeployment activity.

---

## Reduced Deployment Risk

Application deployment and user exposure become separate activities.

---

## Operational Simplicity

The deployment lifecycle remains predictable and easy to manage.

---

# Architecture Summary

Application releases introduce operational risk.

A deployment strategy must therefore protect application availability while enabling continuous delivery of new functionality.

The platform adopts a Blue-Green deployment strategy in which two application environments coexist within the same namespace while consuming a shared MySQL InnoDB Cluster.

The Blue environment represents the current production version, while the Green environment represents the candidate release.

By separating deployment activities from traffic exposure, the platform enables validation, rapid rollback, improved availability, and safer application releases while maintaining operational simplicity.
