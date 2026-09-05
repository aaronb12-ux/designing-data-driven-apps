# Designing Data-Intensive Applications - Chapter 1: Trade-Offs in Data Architecture

## Overview
An application is **data-intensive** when data management is one of the primary challenges in development.

## Core Data Management Needs
Modern applications typically need to:
- **Store data** - databases for persistence
- **Cache results** - speed up expensive operations
- **Search/filter** - indexes for flexible queries
- **Stream processing** - handle real-time events
- **Batch processing** - crunch accumulated data periodically

> **Key insight:** Backend application code is often stateless—after handling an HTTP request, it forgets everything about that request.

---

## Operational vs Analytical Systems

### OLTP (Online Transaction Processing)
**Operational systems** where data is created and modified by application code.

| Characteristic | Details |
|---------------|---------|
| **Read Pattern** | Point queries (fetch individual records by key) |
| **Write Pattern** | CRUD operations on records |
| **Users** | End users via web/mobile apps |
| **Machine Use** | Authorization checks, real-time decisions |
| **Query Types** | Fixed, predefined by application |
| **Data Represents** | Current state (point in time) |
| **Dataset Size** | Gigabytes to terabytes |

### OLAP (Online Analytical Processing)
**Analytical systems** for business intelligence and data science needs.

| Characteristic | Details |
|---------------|---------|
| **Read Pattern** | Aggregate over large number of records |
| **Write Pattern** | Bulk import (ETL) or event streams |
| **Users** | Internal analysts for decision support |
| **Machine Use** | Fraud detection, pattern analysis |
| **Query Types** | Arbitrary queries by analysts |
| **Data Represents** | Historical events over time |
| **Dataset Size** | Terabytes to petabytes |

---

## Data Warehousing & Data Lakes

### Data Warehouse
- Separate database optimized for analytics
- Doesn't affect OLTP operations
- Uses relational model with SQL queries
- Data transformed into specific schema

### Data Lake
- Centralized repository of **raw data** from operational systems
- No transformation into relational schema
- Better for data scientists doing machine learning (SQL limitations)
- Obtained via ETL processes

---

## Systems of Record vs Derived Data

### Systems of Record
- Holds **authoritative/canonical** version of data
- Primary databases where data is written
- Source of truth

### Derived Data Systems
- Result of transforming/processing existing data
- Can be recreated if lost (caches, indexes)
- Analytical systems are usually derived data
- Speed up common read operations

---

## Cloud vs Self-Hosting

### Decision Spectrum
| Control Level | Software | Operations | Example |
|--------------|----------|------------|---------|
| **High** | In-house | In-house | Custom application code |
| **Medium** | Off-the-shelf | In-house | Self-hosted DB on IaaS |
| **Low** | Off-the-shelf | Outsourced | Cloud services/SaaS |

### Cloud Service Trade-offs

**Pros:**
- No need to learn deployment/operations
- Faster time to market
- Automatic scaling and updates

**Cons:**
- No control over missing features
- Dependent on provider uptime
- Difficult to diagnose performance issues
- Must trust provider with data security
- Vendor lock-in risks

---

## Cloud-Native Architecture

Designed to take advantage of cloud services and their unique characteristics.

### System Examples

**Operational/OLTP:**
- Self-hosted: MySQL, PostgreSQL, MongoDB
- Cloud-native: AWS Aurora, Azure SQL DB Hyperscale, Google Cloud Spanner

**Analytical/OLAP:**
- Self-hosted: Teradata, ClickHouse, Spark
- Cloud-native: Snowflake, Google BigQuery, Azure Synapse Analytics

### Key Characteristics

**Layering of Services:**
- Self-hosted: Generic resources (CPU, RAM, filesystem, network)
- Cloud IaaS: Virtual machines with allocated resources
- Higher abstractions: Use-case oriented (good for standard needs)
- Lower abstractions: Build from ground up (for custom requirements)

**Separation of Storage and Compute:**
- Traditional: Same computer handles storage (disk) and computation (CPU/RAM)
- Cloud-native: Storage and compute are **disaggregated**

**Multitenancy:**
- Multiple customers share same hardware
- Benefits: Better utilization, easier scaling, simpler management
- Challenge: Requires careful engineering for performance/security isolation

---

## Operations in the Cloud Era (DevOps/SRE)

Modern operational practices emphasize:
- ✅ **Automation** - repeatable processes over manual jobs
- ✅ **Ephemeral infrastructure** - disposable VMs/services vs long-running servers
- ✅ **Frequent updates** - continuous deployment
- ✅ **Learning from incidents** - blameless postmortems
- ✅ **Knowledge preservation** - documentation survives turnover

---

## Distributed vs Single-Node Systems

### Distributed System
A system where multiple machines communicate via network. Each process is called a **node**.

### Why Go Distributed?

**Inherently Distributed:**
- Multi-user applications (each user on own device)
- Cloud services communicating over network
- Microservices architecture
- Data stored in one service, processed in another

**Fault Tolerance/High Availability:**
- Redundancy across multiple machines
- If one node fails, another takes over
- Enables zero-downtime deployments

---

## Key Takeaways
1. Data-intensive apps face fundamentally different challenges than compute-intensive ones
2. OLTP and OLAP serve different purposes and require different optimizations
3. Cloud-native architecture introduces new patterns (separation of storage/compute, multitenancy)
4. Trade-offs exist at every level: control vs convenience, self-hosted vs managed
5. Modern systems trend toward distribution for reliability and scale
