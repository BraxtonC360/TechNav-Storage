# Architecture

## Overview

The system is a three-layer stack: a physical cluster of Ubuntu Server nodes, a MicroK8s Kubernetes control plane, and an application layer running distributed storage and object store workloads. Everything is on-premises and local-access only — consistent with operating in a compliance-sensitive environment.

```
┌─────────────────────────────────────────────────────────┐
│                    Application Layer                    │
│         Longhorn (block)  ·  MinIO (object / S3)        │
├─────────────────────────────────────────────────────────┤
│                  Kubernetes Layer                       │
│                MicroK8s  ·  RBAC  ·  CoreDNS            │
├─────────────────────────────────────────────────────────┤
│                   Physical Layer                        │
│   Ubuntu Server nodes  ·  Managed switch  ·  VLANs     │
└─────────────────────────────────────────────────────────┘
```

## Physical Layer

Repurposed desktop-class machines running Ubuntu Server LTS. Each node is a dedicated member of the cluster — no shared or dual-purpose workloads.

| Node role | Count | Function |
|---|---|---|
| Control plane | 1 | kube-apiserver, etcd, scheduler, controller-manager |
| Worker | 2+ | Longhorn storage, MinIO, backup ingestion workloads |

All nodes connect to a managed switch. VLANs separate management traffic, data-plane storage traffic, and client-facing backup ingestion — see [networking.md](networking.md).

## Kubernetes Layer

MicroK8s was chosen over vanilla Kubernetes for its simplified lifecycle management on a small cluster: single-binary install, built-in add-on management (`microk8s enable`), and a lower operational surface area for a solo maintainer.

Add-ons in use:

| Add-on | Purpose |
|---|---|
| `dns` | CoreDNS for in-cluster service discovery |
| `helm3` | Application deployment |
| `rbac` | Role-based access control |
| `storage` | Default storage class bootstrap |
| `metrics-server` | Node and pod resource visibility |

Access to the cluster API is local-only; no API server exposure outside the management VLAN.

## Storage Layer

Two storage systems serve different access patterns:

**Longhorn** — distributed block storage. Replicates PersistentVolumes across worker nodes. Provides the underlying PVs that workloads consume. Tolerable to single-node loss without data loss, depending on replica count.

**MinIO** — S3-compatible object store deployed on the cluster. Backup agents (running on client machines or on-cluster) target the MinIO endpoint using standard S3 API calls. This decouples the backup ingestion protocol from the underlying storage implementation — agents write to a well-known API; Longhorn handles the persistence.

```
Backup agent  →  MinIO S3 endpoint  →  Longhorn PV  →  Node local disk
```

## Access Model

| Access type | How |
|---|---|
| Cluster administration | `microk8s kubectl` on control plane node, local network only |
| Storage dashboard | Longhorn UI, NodePort, management VLAN only |
| Backup ingestion | MinIO S3 endpoint, NodePort or LoadBalancer, access VLAN only |
| Data retrieval | Authorized operator via kubectl or MinIO console, scoped by RBAC |

No public internet exposure. No external DNS. No port forwarding.

## Design Rationale

| Decision | Reason |
|---|---|
| MicroK8s over k3s / vanilla | Simpler lifecycle on small cluster; upstream Kubernetes API compatibility |
| Longhorn over local-path | Replication across nodes; survives single-node failure without data loss |
| MinIO over NFS/SMB share | S3 API is lingua franca for backup software; versioning and bucket policies map naturally to retention requirements |
| VLAN segmentation | Separates noisy backup ingestion traffic from management; reduces blast radius on misconfiguration |
| Local-only access | Compliance posture; eliminates a class of external attack surface |
