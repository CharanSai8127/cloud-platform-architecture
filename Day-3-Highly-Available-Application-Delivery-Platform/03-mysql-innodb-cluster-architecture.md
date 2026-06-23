# MySQL InnoDB Cluster Architecture

## Overview

StatefulSets provide the foundational capabilities required for running stateful workloads within Kubernetes.

These capabilities include:

* Stable Identity
* Persistent Storage
* Ordered Startup
* Ordered Shutdown
* Ordered Recovery

While these capabilities are sufficient for deploying stateful applications, they are not sufficient for operating a production database platform.

A database cluster introduces additional operational responsibilities such as:

* Leader Election
* Replication Management
* Health Monitoring
* Failover
* Recovery
* Scaling
* Cluster Reconciliation

These responsibilities require database-specific operational intelligence that Kubernetes does not provide natively.

To address this challenge, the platform adopts the MySQL InnoDB Operator.

---

# Why StatefulSet Alone Is Not Enough

A StatefulSet provides infrastructure primitives for stateful workloads.

It ensures:

```text
mysql-0
mysql-1
mysql-2
```

maintain:

* Stable Identity
* Stable Storage
* Predictable Lifecycle

However, StatefulSet does not understand MySQL.

StatefulSet does not know:

* Which node is Primary
* Which nodes are Replicas
* How replication should occur
* How failover should happen
* How recovery should happen
* How cluster health should be maintained

These responsibilities exist outside the scope of Kubernetes native resources.

As a result, additional operational capabilities are required.

---

# Kubernetes Reconciliation Model

Kubernetes is built around reconciliation.

The platform continuously compares:

```text
Desired State

vs

Actual State
```

When a difference exists between the two states, Kubernetes performs corrective actions to restore the desired state.

Examples include:

* Deployment Controller
* StatefulSet Controller
* DaemonSet Controller

Each controller continuously watches Kubernetes resources and reconciles differences between the desired and actual state of the system.

This reconciliation model is one of the core design principles of Kubernetes.

---

# Kubernetes Extensibility

One of the most powerful capabilities of Kubernetes is extensibility.

Kubernetes is not limited to native resources such as:

* Pods
* Services
* Deployments
* StatefulSets

The platform can be extended to support new resource types through:

* Custom Resource Definitions (CRDs)
* Custom Resources (CRs)
* Controllers

This allows Kubernetes to manage domain-specific platforms and applications using the same reconciliation model.

---

# Custom Resource Definitions

A Custom Resource Definition introduces a new resource type into Kubernetes.

The CRD defines:

* Resource Schema
* Validation Rules
* API Structure

This allows Kubernetes to understand resources that do not exist natively.

For example:

```yaml
apiVersion: mysql.oracle.com/v2
kind: InnoDBCluster
```

After installing the CRD, Kubernetes recognizes the InnoDBCluster resource as a valid API object.

However, defining a schema alone does not perform any operational actions.

Additional logic is required to interpret and manage the resource.

---

# Custom Resources

A Custom Resource represents the desired state of a platform-specific workload.

For a database platform, the custom resource may define:

* Cluster Size
* Replication Configuration
* Storage Requirements
* Routing Configuration
* Availability Requirements

The Custom Resource becomes the declarative representation of the desired database architecture.

Just as Deployments define the desired state of application workloads, Custom Resources define the desired state of database platforms.

---

# Controllers

Controllers continuously observe the state of resources.

Their responsibility is to compare:

```text
Desired State

vs

Actual State
```

and perform corrective actions whenever differences are detected.

Example:

```text
Desired:
3 Database Nodes

Actual:
2 Database Nodes
```

Controller Action:

```text
Create Missing Database Node
```

This continuous reconciliation ensures that the platform remains aligned with the declared configuration.

---

# What Is An Operator?

An Operator extends Kubernetes by combining:

* Custom Resource Definitions
* Custom Resources
* Controllers
* Application-Specific Operational Knowledge

An Operator understands how a specific platform should behave.

Unlike a generic StatefulSet controller, a database operator understands:

* Replication
* Cluster Membership
* Health Monitoring
* Failover
* Recovery
* Scaling

The Operator continuously manages the lifecycle of the database platform using Kubernetes-native reconciliation mechanisms.

---

# Why MySQL InnoDB Operator?

The platform requires more than simply running MySQL containers.

The objective is to operate a reliable database platform capable of supporting application requirements.

The MySQL InnoDB Operator provides database-specific operational intelligence that Kubernetes does not provide natively.

The operator becomes responsible for managing the complete lifecycle of the database cluster.

---

# Database Topology Requirements

Before configuring the operator, the application requirements must first be understood.

The database architecture is driven by:

* Availability Requirements
* Durability Requirements
* Consistency Requirements
* Scalability Requirements

These requirements determine:

* Replication Strategy
* Cluster Topology
* Failover Behavior
* Recovery Expectations

The operator implements these requirements through declarative configuration.

---

# Consensus-Based Database Architecture

The platform adopts a consensus-based clustered architecture.

The cluster consists of multiple coordinated database members.

Example:

```text
MySQL Node 1
MySQL Node 2
MySQL Node 3
```

Rather than operating independently, these members function as a coordinated database platform.

This architecture improves:

* Availability
* Durability
* Failure Recovery
* Operational Stability

while reducing the risk of database outages.

---

# Operator Responsibilities

The MySQL InnoDB Operator continuously manages operational responsibilities including:

---

## Leader Election

The operator manages cluster leadership.

The leader is responsible for coordinating database operations and maintaining cluster consistency.

If the current leader becomes unavailable, the operator coordinates the election of a new leader.

---

## Replication Management

Replication ensures that data is distributed across cluster members.

The operator continuously monitors replication health and ensures that cluster members remain synchronized according to the configured replication strategy.

---

## Health Monitoring

The operator continuously evaluates:

* Cluster Health
* Member Availability
* Replication Status
* Recovery State

This allows failures to be detected automatically.

---

## Failover Management

Infrastructure failures are expected.

Examples include:

* Pod Failure
* Node Failure
* Storage Failure

The operator automatically performs corrective actions to maintain cluster availability.

---

## Promotion And Demotion

Database members may change roles during failover and recovery operations.

The operator manages:

* Promotion Of New Leaders
* Demotion Of Previous Leaders
* Cluster Reconfiguration

without requiring manual intervention.

---

## Recovery

Failed members must be restored safely.

The operator coordinates:

* Rejoining Members
* State Synchronization
* Cluster Recovery

while preserving consistency and availability.

---

## Cluster Reconciliation

The operator continuously reconciles:

```text
Desired Cluster State

vs

Actual Cluster State
```

ensuring that the database platform remains aligned with the intended configuration.

---

# Separation Of Responsibilities

The database platform is built using multiple layers of responsibility.

---

## Kubernetes

Responsible for:

* Scheduling
* Networking
* Storage Integration
* Resource Lifecycle

---

## StatefulSet

Responsible for:

* Stable Identity
* Persistent Storage
* Ordered Lifecycle Management

---

## MySQL InnoDB Operator

Responsible for:

* Cluster Management
* Replication
* Leader Election
* Failover
* Recovery
* Health Monitoring
* Reconciliation

---

## Application

Responsible for:

* Defining Data Requirements
* Availability Expectations
* Consistency Requirements
* Scalability Requirements

This separation ensures that each layer focuses on its intended responsibility.

---

# Architecture Summary

StatefulSet provides the infrastructure foundation required for stateful workloads.

However, operating a production-grade database platform requires additional operational intelligence beyond stable identities and persistent storage.

The MySQL InnoDB Operator extends Kubernetes using Custom Resource Definitions, Custom Resources, and Controllers to continuously reconcile the desired database state with the actual database state.

By automating leader election, replication management, failover, recovery, health monitoring, and cluster reconciliation, the operator becomes the operational brain of the database platform.

This allows the database architecture to satisfy the availability, durability, consistency, and scalability requirements established earlier while remaining aligned with Kubernetes-native operational principles.

