# TechNav Data Infrastructure

Production data storage infrastructure designed, deployed, and operated by a sole technical contributor at a technology services company. The system handles client backup data and business records in a compliance-sensitive environment — built from initial hardware selection through ongoing production operation.

The cluster runs on repurposed Ubuntu Server machines managed by MicroK8s, distributed over a managed switch with VLAN-segmented traffic. No services are exposed to the public internet; all access is local and RBAC-controlled.

## At a Glance

| | |
|---|---|
| **Context** | On-premises production, compliance-sensitive environment |
| **Cluster** | MicroK8s on Ubuntu Server (multi-node) |
| **Storage** | Longhorn distributed block storage · MinIO S3-compatible object store |
| **Networking** | Managed switch · VLAN-segmented · local-access only |
| **Access control** | Kubernetes RBAC · namespace isolation · role-scoped credentials |
| **Contributor** | Sole technical owner — design, procurement, deployment, operations |

## Architecture

Three Ubuntu Server nodes form a MicroK8s cluster. Longhorn provides distributed block storage with replication across nodes; a MinIO deployment exposes an S3-compatible endpoint that backup agents target directly. A managed switch segments traffic by function. See [architecture.md](architecture.md).

## Contents

| Document | What it covers |
|---|---|
| [Architecture](architecture.md) | Cluster topology, storage design, access model |
| [Networking](networking.md) | Switch configuration, VLAN segmentation, local-access design |
| [Storage](storage.md) | Distributed storage, object store, data integrity, retention |
| [Operations](operations.md) | Deployment, monitoring, incident response |
| [Challenges](challenges.md) | Problems encountered and how they were solved |
| [Skills](skills.md) | Skills built — by domain, mapped to production patterns |

## Key Challenges

| Challenge | Outcome |
|---|---|
| [Storage for regulated data](challenges.md#storage-for-regulated-client-data) | Distributed architecture with node-level replication and configurable retention |
| [VLAN segmentation on a single switch](challenges.md#vlan-segmentation-on-a-managed-switch) | Data plane, management, and access traffic isolated on one physical switch |
| [S3-compatible ingestion without cloud dependency](challenges.md#s3-compatible-backup-ingestion-without-cloud) | On-cluster MinIO provides the S3 endpoint; no external dependency or egress cost |
| [Heterogeneous node hardware](challenges.md#heterogeneous-node-hardware) | Addressed via Longhorn replication tolerances and node tagging |
| [Sole contributor operational scope](challenges.md#sole-contributor-operational-scope) | Runbook-driven ops, documented recovery procedures, monitoring with alerting |

## Skills Demonstrated

Kubernetes cluster operations · distributed block storage · S3-compatible object storage · managed switch configuration · VLAN design · Ubuntu Server administration · RBAC and access control · compliance-aware architecture · production ownership · technical documentation

---

*Deployed and operated in a professional context as IT Technician at a technology services company.*
