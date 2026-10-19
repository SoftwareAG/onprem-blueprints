# Adabas & Natural on On-Prem Linux – Resilient HA

## Overview
The Resilient HA blueprint runs Adabas & Natural on on-premises Linux VMs
(RedHat/SUSE) with redundancy across every tier for high availability and fault
tolerance. It is intended for production workloads. Customers provision the VMs,
load balancers, storage, and network shown in the diagram and install the Adabas
(data) and Natural (application) components on them.

## What's New in v3.0.0
- **Natural OpenAPI Server** – exposes Natural subprograms as REST endpoints described
  by OpenAPI. In this blueprint it runs as multiple active-active instances behind
  the HA load balancer.
- **Natural Subprogram Server** – hosts and runs Natural subprograms on behalf of
  remote callers. It is deployed on every Natural application node, so API calls
  survive the loss of a single node.

## Architecture Layers
- **DMZ / External load balancers:** an **HA load balancer** fronts the application
  and API tiers so client traffic (HTTPS) fails over automatically between healthy
  nodes.
- **API tier (Natural OpenAPI Server):** multiple instances of the **Natural OpenAPI
  Server** across Linux VMs publish Natural subprograms as OpenAPI-described REST
  services over **HTTPS**, secured with OIDC.
- **Application tier (Natural):** multiple replicas of the **Natural Availability
  Server** (with NWO, Natural, and Natural Security) running in HA mode across Linux
  VMs. A **Redis Enterprise Cluster** provides the shared session/state cache so the
  replicas operate active-active. Each application node also runs the **Natural
  Subprogram Server**, which serves the subprogram calls coming from the API tier.
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
Five products deliver the continuous availability, API access, and data protection
of this blueprint — two for the data tier, two for the application tier, and one for
the API tier.

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

### Natural Subprogram Server (Application Tier)
The Natural Subprogram Server is the runtime that runs Natural subprograms requested
by remote callers such as the Natural OpenAPI Server.

- **Redundant execution:** an instance runs on each application node, so the API
  tier always has a healthy target if a node is lost.
- **Isolation from online users:** API-driven calls run in their own server
  processes, separate from interactive Natural Availability Server sessions.
- **Elastic capacity:** add application nodes or server processes to scale API
  throughput without changing the application.
- **Consistent security:** Natural Security rules apply to subprograms run through the
  server, just as they do for online users.

### Natural OpenAPI Server (API Tier)
The Natural OpenAPI Server turns existing Natural subprograms into REST APIs without
changes to the Natural code.

- **API-first access:** each exposed subprogram becomes a REST endpoint with an
  OpenAPI specification that client teams can use to generate code and test.
- **Active-active instances:** each API call is handled independently, so every
  instance behind the HA load balancer can serve traffic at the same time.
- **Non-disruptive operations:** take individual instances out of the load balancer
  pool for updates without an API outage.
- **Secure by default:** HTTPS for transport and OIDC for authentication, integrated
  with the customer's identity provider.
- **Integration ready:** can sit behind an existing API gateway for rate limiting,
  versioning, and a developer portal.

### Why It Matters
Together, Adabas Cluster for Linux and the Natural Availability Server remove the two
classic single points of failure in an Adabas & Natural deployment — the database
and the application runtime — while Adabas Encryption for Linux secures the data on
every node. The Natural OpenAPI Server and the Natural Subprogram Server extend the
same availability to API consumers, so Natural business logic can be offered as
always-on REST services alongside the online applications. The result is a
mission-critical platform that supports demanding **RTO/RPO** targets, sustains
**24×7** operations for banking, insurance, government, and healthcare workloads, and
enables planned maintenance and scaling **without downtime**.

## Infrastructure & Security Resources
- **Linux VMs (RedHat/SUSE), multi-node** – redundant API, application, data, and
  management nodes.
- **HA load balancer** – distributes traffic to healthy Natural OpenAPI Server and
  Natural nodes in the DMZ.
- **Natural OpenAPI Server** – multiple active-active instances publishing Natural
  subprograms as REST APIs.
- **Natural Subprogram Server** – one instance per application node, running
  subprograms called through the API tier.
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
  HTTPS to the API tier only from approved sources, Adabas ports (ADATCP/S) reachable
  only from the Natural layer, management access restricted to trusted ranges.
- **TLS certificates** – issued by the corporate PKI for the HTTPS endpoints.
- **Site-to-site connectivity** – MPLS / SD-WAN / VPN between the office edge router
  and the datacenter edge router.
- **Shared infrastructure services** – Git, OIDC, LDAP/S, Active Directory,
  infrastructure backup, file server, log server, and monitoring & alerting.

## Architecture Diagrams
See the `docs` folder (v3.0.0 diagrams to be added):
- `Adabas and Natural on On-Prem - ResilientHA.gif`

## Intended Use
Production workloads requiring high availability, business continuity, disaster
recovery, and highly available API access to Natural business logic. For simpler
development setups use the Basic blueprint; for container-native deployments use the
container blueprints (Docker Compose, Helm, Operator).
