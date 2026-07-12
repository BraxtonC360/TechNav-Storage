# Stack

## Hardware

| Component | Role |
|---|---|
| Desktop-class machines (repurposed) | Cluster nodes — Ubuntu Server LTS |
| Managed network switch | VLAN-segmented cluster backbone |
| Local workstation | Operator access (management VLAN) |

## Operating System

| Software | Version | Notes |
|---|---|---|
| Ubuntu Server | LTS (current) | All nodes; minimal install |

## Kubernetes

| Software | Notes |
|---|---|
| MicroK8s | Lightweight K8s distribution; snap-managed; upstream-compatible API |
| CoreDNS | In-cluster DNS (via `microk8s enable dns`) |
| Helm 3 | Application deployment |
| metrics-server | Node and pod resource visibility |

## Storage

| Software | Role |
|---|---|
| Longhorn | Distributed block storage; PersistentVolume provider |
| MinIO | S3-compatible object store; backup ingestion endpoint |

## Networking

| Component | Notes |
|---|---|
| Managed switch | CLI-configured; VLAN trunk/access port segmentation |
| VLANs | Management · Data plane · Ingestion |

## Access Control

| Mechanism | Scope |
|---|---|
| Kubernetes RBAC | ServiceAccounts, Roles, RoleBindings per workload |
| MinIO access keys | Per-client, per-bucket scoped credentials |
| SSH key auth | Node access; password auth disabled |
| Network segmentation | VLAN-enforced access boundaries |

## Deployment Method

All workloads deployed via Helm with version-controlled values files. Cluster configuration managed via `microk8s` CLI and `kubectl`. Switch configuration exported and documented after each change.
