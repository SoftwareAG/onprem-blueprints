# Adabas & Natural on On-Prem Linux – Basic

## Overview
The Basic blueprint runs Adabas & Natural on on-premises Linux VMs (RedHat/SUSE). It
is intended for development, prototyping, and lift-and-shift workloads. Customers
provision the VMs, storage, and network shown in the diagrams and install the Adabas
(data) and Natural (application) components on them.

## Architecture Layers
- **Application layer (Natural):** the Natural runtime and applications — Natural,
  Natural Batch, and NDV (Natural Development Server) — reachable by end users and
  developer tooling.
- **Data layer (Adabas):** the Adabas database with Adabas Encryption for Linux
  (AEL), the Adabas REST Server, and Adabas Manager. Database containers
  (ASSO · DATA · WORK · PLOG) are held on SAN / block storage.

The Basic blueprint supports two shapes:
- **Development – Single Instance:** both layers co-located on one Linux VM for
  simplicity. Users connect via terminal emulation over **SSH**, NaturalONE / NJX
  over **SSL/TLS**, VS Code, and browsers over **HTTPS**.
- **Distributed:** the Natural application layer and the Adabas data layer run on
  separate Linux VMs so each layer can be sized, secured, and maintained
  independently. The layers communicate over **ADATCP/S**.

## Infrastructure & Security Resources
- **Linux VMs (RedHat/SUSE)** – compute hosting the application and data layers.
- **SAN / Block Storage** – persistent storage for the Adabas containers
  (ASSO · DATA · WORK · PLOG); enable encryption at rest.
- **Adabas Encryption for Linux (AEL)** – encryption of Adabas data.
- **Firewall** – boundary between the customer network and the on-prem datacenter;
  restrict SSH and management access to trusted ranges and allow the Adabas ports
  (ADATCP/S) only from the Natural layer.
- **Site-to-site connectivity** – MPLS / SD-WAN / VPN between the office edge router
  and the datacenter edge router.
- **Shared infrastructure services** – Git / GitHub, OIDC, LDAP/S, Active Directory,
  infrastructure backup, file server, log server, and monitoring & alerting.

## Architecture Diagrams
See the `docs` folder:
- `Adabas and Natural on On-Prem - Basic Development.gif`
- `Adabas and Natural on On-Prem - Basic Distributed.gif`

## Intended Use
Development, prototyping, and small production workloads. For high availability and
fault tolerance, use the Resilient HA blueprint; for container-native deployments,
use the container blueprints (Docker Compose, Helm, Operator).
