# Adabas & Natural on On-Prem Linux – Resilient HA

## Overview
The Resilient HA blueprint runs Adabas & Natural on on-premises Linux VMs
(RedHat/SUSE) with redundancy across every tier for high availability and fault
tolerance. It is intended for production workloads. Customers provision the VMs,
load balancers, storage, and network shown in the diagram and install the Adabas
(data) and Natural (application) components on them.

## Architecture Layers
- **DMZ / External load balancers:** an **HA load balancer** fronts the application
  tier so client traffic (HTTPS) fails over automatically between healthy nodes.
- **Application tier (Natural):** multiple replicas of the **Natural Availability
  Server** (with NWO, Natural, and Natural Security) running in HA mode across Linux
  VMs. A **Redis Enterprise Cluster** provides the shared session/state cache so the
  replicas operate active-active.
- **Data tier (Adabas):** a 3-node **Adabas Cluster for Linux** — a **shared-nothing**
  database cluster. A **Primary** node (Adabas with AEL, Adabas REST Server, Natural
  Batch) replicates to two **Secondary** nodes for resilience. Each node owns its own
  dedicated SAN / block storage (ASSO · DATA · WORK · PLOG) — nothing is shared
  between nodes — with read/write on the primary and read replicas on the
  secondaries.
- **Management tier:** a dedicated Linux VM running **Adabas Manager** for
  administration of the data tier.

The application and data tiers communicate over **ADATCP/S**.

## Key High Availability Products
Three purpose-built products deliver the continuous availability and data protection
of this blueprint — two for the data tier and one for the application tier.

### Adabas Cluster for Linux (Data Tier)
Adabas Cluster for Linux is a **3-node, shared-nothing database cluster**. Each node
owns its own storage and an independent copy of the data, kept in sync through
replication rather than a shared disk. This removes the shared storage subsystem as a
single point of failure and is the foundation of the primary/secondary data tier in
this blueprint.

- **No single point of failure:** shared-nothing means there is no shared disk or
  shared cache to fail; if a node or its storage is lost, the remaining nodes keep
  serving requests.
- **Automatic failover & recovery:** on node failure, surviving nodes take over and
  in-flight work is recovered automatically — the database stays online.
- **Continuous availability for maintenance:** patch, upgrade, and perform hardware
  maintenance node-by-node with **zero planned downtime** (rolling maintenance).
- **Read scalability:** secondary nodes serve read workloads, offloading the primary
  and improving throughput for growing workloads.
- **Data resilience:** each node keeps an independent, encrypted copy of the
  database, protecting against both storage-level and node-level failures.

### Adabas Encryption for Linux / AEL (Data Tier)
Adabas Encryption for Linux provides **transparent encryption of Adabas data at
rest** on every node of the cluster, complementing the availability that Adabas
Cluster for Linux delivers.

- **Data-at-rest protection:** the Adabas containers (ASSO · DATA · WORK · PLOG) are
  encrypted on each shared-nothing node, so every independent copy of the database is
  protected.
- **Transparent to applications:** no changes to Natural or application code — the
  Adabas nucleus handles encryption/decryption inline.
- **Compliance enablement:** helps meet data-protection and regulatory mandates
  (e.g. GDPR, PCI-DSS, HIPAA) for sensitive workloads.
- **Defense in depth:** combined with network isolation and encryption in transit
  (ADATCP/S), it protects data across its full lifecycle.

### Natural Availability Server (Application Tier)
The Natural Availability Server is the web front end that makes Natural online
applications highly available. It provides a Natural terminal emulator as a modern
Angular and REST-based web application, and it lets Natural online applications run
in a scalable, highly available environment. Depending on the infrastructure and the
architecture chosen, availability can exceed **99.99%**. It runs as **multiple
replicas in HA mode**, sharing session and application state through a **Redis
cache** so that all replicas act as **one logical, always-on application service**
behind the load balancer.

- **Transparent failover:** session and application state are held in the Redis
  cache, so if a replica is lost, users are served by another replica without
  interruption.
- **Active-active replicas:** all Natural Availability Server replicas serve traffic
  simultaneously, maximizing utilization and eliminating idle standby servers.
- **Elastic scale-out:** add replicas behind the load balancer to grow capacity
  linearly as demand increases.
- **Non-disruptive operations:** drain and update individual replicas for
  application or platform maintenance without a service outage.
- **Consistent security:** Natural Security is enforced uniformly across every node
  in the cluster.

### Why It Matters
Together, Adabas Cluster for Linux and the Natural Availability Server remove the two
classic single points of failure in an Adabas & Natural deployment — the database
and the application runtime — while Adabas Encryption for Linux secures the data on
every node. The result is a mission-critical platform that supports demanding
**RTO/RPO** targets, sustains **24×7** operations for banking, insurance,
government, and healthcare workloads, and enables planned maintenance and scaling
**without downtime**.

## Infrastructure & Security Resources
- **Linux VMs (RedHat/SUSE), multi-node** – redundant application, data, and
  management nodes.
- **HA load balancer** – distributes traffic to healthy Natural nodes in the DMZ.
- **Natural Availability Server** – multiple replicas in HA mode forming an
  active-active Natural application tier with transparent session failover.
- **Redis Enterprise Cluster** – shared session/state cache for the active-active
  Natural Availability Server replicas.
- **Adabas Cluster for Linux** – 3-node shared-nothing database cluster with a
  primary and two secondary replicas.
- **SAN / Block Storage (per node)** – dedicated, encrypted persistent storage for
  the Adabas containers (ASSO · DATA · WORK · PLOG) on each node.
- **Adabas Encryption for Linux (AEL)** – transparent encryption of Adabas data at
  rest on every cluster node.
- **Firewall** – boundary between the customer network and the on-prem datacenter;
  Adabas ports (ADATCP/S) reachable only from the Natural layer, management access
  restricted to trusted ranges.
- **Site-to-site connectivity** – MPLS / SD-WAN / VPN between the office edge router
  and the datacenter edge router.
- **Shared infrastructure services** – Git, OIDC, LDAP/S, Active Directory,
  infrastructure backup, file server, log server, and monitoring & alerting.

## Architecture Diagrams
See the `docs` folder:
- `Adabas and Natural on On-Prem - ResilientHA.gif`

## Intended Use
Production workloads requiring high availability, business continuity, and disaster
recovery. For simpler development setups use the Basic blueprint; for
container-native deployments use the container blueprints (Docker Compose, Helm,
Operator).
