# Adabas & Natural on On-Prem Linux – Basic

## Overview
The Basic blueprint runs Adabas & Natural on on-premises Linux VMs (RedHat/SUSE). It
is intended for development, prototyping, and lift-and-shift workloads. Customers
provision the VMs, storage, and network shown in the diagrams and install the Adabas
(data) and Natural (application) components on them.

## What's New in v3.0.0
- **Natural OpenAPI Server** – exposes Natural subprograms as REST endpoints described
  by OpenAPI, so modern applications, portals, and integration platforms can call
  Natural business logic over HTTPS.
- **Natural Subprogram Server** – hosts and runs Natural subprograms on behalf of
  remote callers such as the Natural OpenAPI Server, separating API traffic from
  interactive Natural sessions.

## Architecture Layers
- **API layer (Natural OpenAPI Server):** publishes selected Natural subprograms as
  OpenAPI-described REST services over **HTTPS**, secured with OIDC.
- **Application layer (Natural):** the Natural runtime and applications — Natural,
  Natural Security, Natural Batch, NDV (Natural Development Server), the **Natural
  Subprogram Server** and, in the Distributed shape, the Natural Availability Server
  (NAS) with Natural Web I/O (NWO) — reachable by end users, API callers, and
  developer tooling.
- **Data layer (Adabas):** the Adabas database with Adabas Encryption for Linux
  (AEL), the Adabas REST Server, and Adabas Manager. Database containers
  (ASSO · DATA · WORK · PLOG) are held on SAN / block storage.

The Basic blueprint supports two shapes:
- **Development – Single Instance:** all layers co-located on one Linux VM for
  simplicity. Users connect via terminal emulation over **SSH**, NaturalONE / NJX
  over **SSL/TLS**, VS Code and browsers over **HTTPS**, and API clients call the
  Natural OpenAPI Server over **HTTPS**.
- **Distributed:** the Natural application layer and the Adabas data layer run on
  separate Linux VMs so each layer can be sized, secured, and maintained
  independently. The application VM runs the Natural OpenAPI Server, the Natural
  Subprogram Server, the Natural Availability Server (browser users over **HTTPS**,
  rendered through NWO), NDV, Natural, and Natural Security. The data VM runs Adabas
  (AEL), the Adabas REST Server, Adabas Manager, and Natural Batch. The layers
  communicate over **ADATCP/S**. A single NAS instance needs no shared session store;
  for multiple NAS replicas and failover, use the Resilient HA blueprint.

## Key New Products

### Natural OpenAPI Server
The Natural OpenAPI Server turns existing Natural subprograms into REST APIs without
changes to the Natural code.

- **API-first access:** each exposed subprogram becomes a REST endpoint with an
  OpenAPI specification that client teams can use to generate code and test.
- **No code changes:** existing business logic stays in Natural. The server maps the
  subprogram parameters to JSON request and response bodies.
- **Secure by default:** HTTPS for transport and OIDC for authentication, integrated
  with the customer's identity provider (e.g. Active Directory federation, Keycloak).
- **Integration ready:** can sit behind an existing API gateway or reverse proxy for
  rate limiting, versioning, and a developer portal.

### Natural Subprogram Server
The Natural Subprogram Server is the runtime that runs Natural subprograms requested
by remote callers.

- **Dedicated execution:** runs API-driven subprogram calls in their own server
  processes, separate from interactive Natural sessions.
- **Direct data access:** runs on the Natural application layer and reaches Adabas
  over **ADATCP/S**, like any other Natural workload.
- **Independent sizing:** the number of server processes can be tuned to API demand
  without affecting online users.

## Infrastructure & Security Resources
- **Linux VMs (RedHat/SUSE)** – compute hosting the API, application, and data
  layers.
- **SAN / Block Storage** – persistent storage for the Adabas containers
  (ASSO · DATA · WORK · PLOG); enable encryption at rest.
- **Adabas Encryption for Linux (AEL)** – encryption of Adabas data.
- **Firewall** – boundary between the customer network and the on-prem datacenter;
  restrict SSH and management access to trusted ranges, allow HTTPS to the Natural
  OpenAPI Server only from approved clients, and allow the Adabas ports (ADATCP/S)
  only from the Natural layer.
- **TLS certificates** – issued by the corporate PKI for the Natural OpenAPI Server
  HTTPS endpoint.
- **Site-to-site connectivity** – MPLS / SD-WAN / VPN between the office edge router
  and the datacenter edge router.
- **Shared infrastructure services** – Git / GitHub, OIDC, LDAP/S, Active Directory,
  infrastructure backup, file server, log server, and monitoring & alerting.

## Architecture Diagrams
See the `docs` folder. The diagrams use the same on-prem notation as the Resilient HA
v3.0.0 blueprint (the editable `.drawio` sources are kept locally and are not
published):
- **Development** – `Adabas and Natural on On-Prem - Basic Development-v3.0.0.gif` –
  one Linux VM hosting the Natural OpenAPI Server, Natural Subprogram Server, NDV,
  Natural, Natural Security, Natural Batch, and the Adabas components.
- **Distributed** – `Adabas and Natural on On-Prem - Basic Distributed-v3.0.0.gif` –
  an application VM (API and Natural runtime) and a data VM (Adabas and Natural
  Batch) connected over ADATCP/S.

## Intended Use
Development, prototyping, API enablement pilots, and small production workloads. For
high availability and fault tolerance, use the Resilient HA blueprint; for
container-native deployments, use the container blueprints (Docker Compose, Helm,
Operator).
