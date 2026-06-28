# Azure SQL Platform Architecture

## Overview

Data is one of the most valuable assets within an enterprise application. While Kubernetes excels at managing stateless workloads, managing relational databases introduces an entirely different set of operational challenges. Databases must preserve business data while simultaneously providing high availability, durability, consistency, security, and disaster recovery capabilities.

Organizations therefore increasingly adopt Platform as a Service (PaaS) database offerings rather than self-managing database infrastructure inside Kubernetes clusters. Azure SQL Database abstracts the operational complexity of database management while allowing organizations to focus on application development and business requirements.

This project adopts Azure SQL Database as the primary data platform, integrating it with Azure Kubernetes Service (AKS) through Private Endpoints, Microsoft Entra ID, and Azure-native high availability capabilities.

---

# Why Azure SQL Database?

One of the first architectural decisions when designing a cloud-native platform is determining where application data should reside.

There are two common approaches.

## Self-Managed Database

Deploying a database inside Kubernetes provides complete control over the database environment.

However, organizations become responsible for:

- Infrastructure Provisioning
- Operating System Maintenance
- Database Installation
- Security Patching
- Backup Management
- High Availability
- Disaster Recovery
- Monitoring
- Performance Tuning

As the platform grows, the operational effort required to maintain the database increases significantly.

---

## Managed Platform Database

Azure SQL Database shifts infrastructure responsibilities to Microsoft.

Microsoft becomes responsible for:

- Infrastructure Availability
- Operating System Maintenance
- Database Engine Updates
- Security Patching
- Platform Monitoring
- Automatic High Availability
- Service Reliability

The organization remains responsible for:

- Database Schema
- Data Security
- User Access
- Performance Optimization
- Application Logic

This follows Azure's Shared Responsibility Model.

Rather than managing database infrastructure, the platform consumes database capabilities as a managed cloud service.

---

# Enterprise Database Requirements

Modern enterprise applications require more than simply storing information.

The database platform must satisfy several architectural requirements.

These include:

- Availability
- Durability
- Consistency
- Scalability
- Security
- Business Continuity
- Disaster Recovery
- Operational Simplicity

Azure SQL Database provides these capabilities while significantly reducing operational overhead.

---

# Azure SQL As A Platform Service

Unlike databases deployed inside Kubernetes, Azure SQL Database operates as a managed cloud service.

The application does not communicate with a database container running inside the cluster.

Instead, requests are sent to a fully managed Azure service.

```text
.NET Application
        │
        ▼
Azure SQL Database
```

This architectural model separates application lifecycle from database lifecycle.

Application deployments, Kubernetes upgrades, and node replacements no longer directly impact database infrastructure.

---

# Database Provisioning Architecture

The Terraform module provisions the Azure SQL platform by creating the necessary database infrastructure before the application is deployed.

The architecture includes:

- Azure SQL Server
- Azure SQL Database
- Primary Region
- Secondary Region
- Failover Group
- Private Endpoint
- Diagnostic Settings
- Log Analytics Integration

Each resource contributes to the overall reliability and security of the data platform.

---

# Primary And Secondary Database Architecture

Business-critical applications cannot depend on a single database instance.

Unexpected failures such as infrastructure outages, regional disruptions, or service interruptions should not permanently impact application availability.

To improve resilience, the platform provisions both:

- Primary Database
- Secondary Database

The primary database services application traffic during normal operation.

The secondary database continuously receives replicated changes from the primary.

This architecture supports business continuity while minimizing service disruption.

---

# Geo-Replication

Azure SQL Database supports asynchronous geo-replication between regions.

Every committed transaction on the primary database is replicated to the secondary database.

This provides:

- Disaster Recovery
- Geographic Redundancy
- Improved Availability
- Regional Fault Tolerance

Geo-replication ensures that application data remains available even if the primary region experiences an outage.

---

# Failover Groups

Replication alone does not guarantee application continuity.

Applications must also know which database instance should receive requests.

Azure SQL Failover Groups provide a logical abstraction over primary and secondary databases.

Instead of connecting directly to a specific database server, applications communicate with the failover endpoint.

```text
Application
        │
        ▼
Failover Group
        │
        ▼
Primary Database
```

If the primary becomes unavailable, Azure automatically redirects client connections to the secondary database.

Applications continue communicating through the same endpoint without requiring configuration changes.

---

# Secure Database Connectivity

By default, Azure SQL Database exposes a public endpoint.

Although functional, exposing business-critical databases to the public internet increases the attack surface.

This project adopts Azure Private Endpoints to eliminate unnecessary public exposure.

The communication flow becomes:

```text
AKS
        │
        ▼
Private Endpoint
        │
        ▼
Azure SQL Database
```

Traffic remains entirely within the Azure backbone network.

No database communication traverses the public internet.

---

# Identity-Based Database Access

Network connectivity alone does not authorize access to the database.

Authentication remains a separate concern.

Azure SQL Database integrates with Microsoft Entra ID, allowing trusted identities to securely authenticate without distributing static credentials.

Application workloads authenticate using Azure Workload Identity, ensuring that database access is granted only to authorized workloads.

This establishes a layered security model where:

- Networking controls communication.
- Identity controls authentication.
- Authorization controls database access.

---

# Monitoring And Diagnostics

Operating a production database requires continuous visibility into platform health.

The Terraform module integrates Azure SQL Database with Azure Monitor and Log Analytics.

Monitoring enables:

- Performance Analysis
- Query Diagnostics
- Resource Utilization
- Failure Detection
- Operational Auditing

These capabilities improve operational awareness and support proactive platform management.

---

# Database Communication Flow

The complete communication path becomes:

```text
.NET Application
        │
        ▼
Kubernetes Service
        │
        ▼
AKS Node
        │
        ▼
Private Endpoint
        │
        ▼
Azure SQL Database
        │
        ▼
Geo Replication
        │
        ▼
Secondary Database
```

Application workloads communicate securely while Azure manages replication and business continuity.

---

# Advantages Of A Managed Database Platform

Adopting Azure SQL Database provides several operational benefits.

These include:

- Reduced Operational Complexity
- Managed High Availability
- Automatic Patching
- Built-in Disaster Recovery
- Geo-Replication
- Failover Automation
- Centralized Monitoring
- Enterprise Security
- Integration With Microsoft Entra ID

Rather than investing engineering effort into operating database infrastructure, teams can focus on application functionality.

---

# Trade-Offs

Using a managed cloud database introduces several trade-offs.

Advantages:

- Simplified Operations
- Enterprise Reliability
- Managed High Availability
- Reduced Maintenance
- Built-in Monitoring

Trade-offs:

- Network Latency Compared To In-Cluster Databases
- Reduced Infrastructure Customization
- Cloud Provider Dependency
- Consumption-Based Pricing
- External Network Dependency

The platform intentionally accepts these trade-offs because the operational advantages significantly outweigh the additional network latency and infrastructure abstraction.

---

# Summary

Azure SQL Database serves as the managed data platform for this architecture.

Rather than deploying and operating relational databases within Kubernetes, the platform leverages Azure-native database services to provide enterprise-grade availability, durability, scalability, and operational simplicity.

Private Endpoints secure communication, Microsoft Entra ID enables identity-based authentication, geo-replication and Failover Groups provide business continuity, while Azure Monitor delivers operational visibility.

This approach allows the application platform to consume a reliable, secure, and highly available database service while minimizing operational complexity and aligning with modern cloud-native architectural practices.
