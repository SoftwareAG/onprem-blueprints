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
- **Application tier (Natural):** multiple Linux VMs, each running NHA (Natural High
  Availability), NWO, Natural, and Natural Security. A **Redis Enterprise Cluster**
  provides shared session/state so the Natural nodes operate active-active.
- **Data tier (Adabas):** a **Primary** Adabas VM (Adabas with AEL, Adabas REST
  Server, Natural Batch) replicating to two **Secondary** Adabas VMs for database
  resilience. Each node has its own SAN / block storage (ASSO · DATA · WORK · PLOG),
  with read/write on the primary and read replicas on the secondaries.
- **Management tier:** a dedicated Linux VM running **Adabas Manager** for
  administration of the data tier.

The application and data tiers communicate over **ADATCP/S**.

## Infrastructure & Security Resources
- **Linux VMs (RedHat/SUSE), multi-node** – redundant application, data, and
  management nodes.
- **HA load balancer** – distributes traffic to healthy Natural nodes in the DMZ.
- **Redis Enterprise Cluster** – shared state for active-active Natural nodes.
- **Adabas replication** – primary/secondary replication across data-tier nodes.
- **SAN / Block Storage (per node)** – encrypted persistent storage for the Adabas
  containers (ASSO · DATA · WORK · PLOG).
- **Adabas Encryption for Linux (AEL)** – encryption of Adabas data.
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
