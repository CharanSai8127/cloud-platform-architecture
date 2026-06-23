# Traffic Management And Gateway API

## Overview

Blue-Green deployment introduces two independent application versions that coexist simultaneously.

While the deployment strategy allows a new version to be deployed safely, the platform still requires a mechanism to determine which version receives production traffic.

Deploying a new application version and exposing it to users are two separate activities.

A deployment should not automatically become user-facing.

Instead, traffic should be controlled independently so that:

- New Versions Can Be Validated
- Production Availability Is Preserved
- Rollbacks Can Be Performed Quickly
- Application Releases Remain Safe

This document focuses on the traffic management architecture that enables Blue-Green deployments within the platform.

---

# Why Traffic Management Is Important

Application deployments introduce risk.

Examples include:

- Startup Failures
- Configuration Errors
- Dependency Issues
- Runtime Failures

If traffic is automatically routed to a newly deployed version:

```text
Deploy New Version
        ↓
Traffic Arrives
        ↓
Failure
        ↓
User Impact
```

the deployment process itself becomes a reliability risk.

To reduce this risk, traffic management must be separated from application deployment.

The platform should be capable of:

- Deploying New Versions
- Validating New Versions
- Controlling Traffic Exposure
- Reverting Traffic Quickly

without requiring additional deployments.

---

# Application Traffic Flow

The application traffic follows a controlled path through the platform.

```text
User
 ↓
DNS
 ↓
AWS Load Balancer
 ↓
Gateway API
 ↓
HTTPRoute
 ↓
Blue Service / Green Service
 ↓
Application Pods
 ↓
MySQL InnoDB Cluster
```

Each component has a specific responsibility.

---

## DNS

DNS provides a user-friendly entry point into the platform.

Users access the application through a domain name rather than infrastructure-specific addresses.

---

## AWS Load Balancer

The load balancer exposes the platform externally and forwards traffic into the Kubernetes cluster.

Its responsibility is to provide a stable external entry point.

---

## Gateway API

The Gateway acts as the traffic entry point inside the cluster.

It receives incoming requests and applies routing policies defined by the platform.

---

## HTTPRoute

HTTPRoute determines where traffic should be sent.

This is the component responsible for directing traffic toward the appropriate application version.

---

## Application Services

The Blue and Green services expose the corresponding application deployments.

Traffic is forwarded to the active version selected by the routing configuration.

---

## MySQL InnoDB Cluster

Both application versions consume the same database platform.

The database remains the authoritative source of business data regardless of which application version is active.

---

# Why Gateway API?

Modern application platforms require more than simple traffic forwarding.

Traffic management must support:

- Version Transitions
- Application Isolation
- Controlled Exposure
- Future Scalability

The platform adopts Gateway API because it provides a Kubernetes-native approach to traffic management.

Gateway API introduces clearer separation between:

- Infrastructure Ownership
- Traffic Policies
- Application Routing

This separation improves maintainability and operational clarity.

---

# Gateway Resource

The Gateway resource acts as the entry point for application traffic.

Its responsibilities include:

- Accepting Incoming Traffic
- Defining Listeners
- Exposing Application Endpoints
- Delegating Routing Decisions

Example:

```text
Internet
        ↓
Gateway
```

The Gateway does not decide which application receives traffic.

Instead, it provides the entry point through which routing decisions are applied.

---

# HTTPRoute Resource

HTTPRoute defines how traffic is routed after it enters the Gateway.

Its responsibilities include:

- Defining Routing Rules
- Selecting Backends
- Mapping Requests To Services

Example:

```text
Gateway
        ↓
HTTPRoute
        ↓
Application Service
```

HTTPRoute becomes the primary mechanism used to control production traffic.

---

# Blue-Green Traffic Switching

The platform maintains two independent application environments:

```text
Blue Deployment

Green Deployment
```

Each deployment is exposed through its own service.

Example:

```text
Blue Service

Green Service
```

The HTTPRoute determines which service receives production traffic.

---

## Initial State

The currently active version receives traffic.

```text
Gateway
        ↓
HTTPRoute
        ↓
Blue Service
        ↓
Blue Pods
```

Blue acts as the production environment.

Green remains deployed but inactive.

---

## New Version Deployment

A new version is deployed into Green.

```text
Blue
        ↓
Production

Green
        ↓
Candidate Release
```

At this stage:

- Users Continue Using Blue
- Green Receives No Production Traffic
- Validation Activities Can Begin

---

## Traffic Promotion

Once Green has been validated:

```text
Gateway
        ↓
HTTPRoute
        ↓
Green Service
        ↓
Green Pods
```

Traffic is redirected to Green.

The deployment itself does not change.

Only the routing decision changes.

This separation reduces deployment risk.

---

# Rollback Through Routing

Failures are expected.

The platform must provide a fast recovery mechanism.

If issues are detected after Green becomes active:

```text
Gateway
        ↓
Green
        ↓
Issue Detected
```

Traffic can be redirected back to Blue.

Example:

```text
Gateway
        ↓
HTTPRoute
        ↓
Blue Service
```

The previous version remains available throughout the process.

This allows recovery without:

- Rebuilding Images
- Redeploying Applications
- Waiting For New Pods

Rollback becomes a routing operation rather than a deployment operation.

---

# Shared Namespace Routing Model

The platform adopts a shared namespace architecture.

Within the namespace:

```text
Application Namespace

├── Blue Deployment
├── Green Deployment
├── Blue Service
├── Green Service
└── HTTPRoute
```

Both application versions coexist simultaneously.

The HTTPRoute determines which service receives traffic.

This approach provides:

- Operational Simplicity
- Reduced Resource Duplication
- Faster Version Transitions

while maintaining deployment safety.

---

# Separation Of Responsibilities

The traffic management architecture intentionally separates responsibilities.

---

## Application Deployments

Responsible for:

- Running Application Code
- Processing Requests
- Consuming Data

---

## Gateway

Responsible for:

- Accepting Traffic
- Providing Entry Points

---

## HTTPRoute

Responsible for:

- Routing Decisions
- Version Selection
- Traffic Switching

---

## Database Platform

Responsible for:

- Data Availability
- Durability
- Consistency
- Replication

This separation allows each layer to evolve independently.

---

# Traffic Management Benefits

The architecture provides several operational advantages.

---

## Improved Availability

Application deployments no longer directly impact production traffic.

Users continue interacting with the active version until promotion occurs.

---

## Safer Releases

New versions can be deployed and validated before receiving traffic.

---

## Faster Recovery

Rollback becomes a traffic-routing operation.

Recovery time is significantly reduced.

---

## Operational Simplicity

Traffic management remains declarative and predictable.

The routing configuration determines the active version.

---

## Version Isolation

Blue and Green environments can coexist without impacting each other until traffic promotion occurs.

---

# Architecture Summary

Blue-Green deployment requires a mechanism for controlling production traffic independently from application deployment.

The platform adopts Gateway API and HTTPRoute resources to provide this capability.

Traffic enters through the Gateway, while HTTPRoute determines which application version receives requests.

By separating deployment activities from traffic exposure, the platform enables:

- Safer Releases
- Controlled Validation
- Rapid Rollback
- Improved Availability
- Operational Simplicity

This architecture allows application versions to be deployed, validated, promoted, and reverted without modifying the application itself, creating a reliable foundation for continuous application delivery.
