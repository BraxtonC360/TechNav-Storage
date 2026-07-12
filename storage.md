# Storage

## Requirements

The storage system was designed around three non-negotiable properties:

1. **Durability** — data written must survive a single-node failure without loss.
2. **Retrievability** — data must be recoverable on-demand; write-only backup systems fail when you need them most.
3. **Retention compliance** — data must be held for configurable periods and protected from premature deletion.

## Two-Layer Architecture

Storage is split by access pattern: block storage for the cluster's internal workloads and persistent state, object storage for backup data ingestion.

### Longhorn — Distributed Block Storage

Longhorn runs as a DaemonSet across all worker nodes. Each PersistentVolumeClaim gets a replicated volume distributed across nodes according to the configured replica count.

Key properties:
- **Replication** — PV data is replicated across `N` nodes; survives `N-1` node failures without loss
- **Volume snapshots** — point-in-time snapshots of PVs, schedulable via recurring job
- **UI** — Longhorn dashboard (NodePort, management VLAN) provides volume health, replica state, and snapshot management at a glance
- **Node tagging** — nodes tagged by storage capacity and hardware generation; Longhorn scheduling respects tags to avoid placing replicas on underpowered nodes

Longhorn backs all cluster-internal persistent state — MinIO's data directory, any stateful workloads, and Kubernetes-native backup tooling storage.

### MinIO — S3-Compatible Object Store

MinIO is deployed on-cluster and provides an S3-compatible API endpoint. Backup agents on client machines (and other internal systems) write to MinIO using standard S3 API calls — the same interface they'd use against AWS S3 or any other S3-compatible target.

This separation matters: the backup ingestion interface is a stable, well-known API. If the underlying storage implementation changes (e.g., migrating Longhorn to a different backend), the agents don't change.

Key properties:
- **Bucket-per-client** — each data source gets an isolated bucket; a misconfigured client cannot overwrite another client's data
- **Object locking** — WORM (Write Once, Read Many) locking on retention-sensitive buckets prevents modification or deletion before the retention period expires
- **Versioning** — object versioning enabled; delete operations create a delete marker rather than removing data, supporting recovery from accidental deletion
- **Access keys** — per-client access keys scoped to individual buckets; no client key can reach another bucket

### Data Flow

```
Client machine
    │
    │  S3 PUT (backup agent)
    ▼
MinIO endpoint (NodePort, ingestion VLAN)
    │
    │  persists to
    ▼
Longhorn PersistentVolume
    │
    │  replicated across
    ▼
Worker node local disks (N replicas)
```

## Data Integrity

A backup system that hasn't been tested for recovery is not a backup system.

- **MinIO data integrity** — MinIO checksums objects on write and verifies on read (bitrot detection)
- **Restore testing** — periodic spot-restores from backup buckets to verify recoverability; documented in the operations runbook
- **Volume health checks** — Longhorn volume health is monitored; degraded volumes (replica count below threshold) trigger alerts before they reach a critical state

## Retention

Retention is implemented at the bucket level using MinIO object lifecycle policies. Lifecycle rules:
- Delete objects older than the configured retention period automatically
- WORM locks prevent deletion of objects before the minimum retention period, regardless of operator action

Retention periods are set per-bucket according to data classification. No retention periods or data classifications are documented here.

## Skills This Layer Built

Distributed storage design · PersistentVolume and StorageClass configuration · MinIO deployment and administration · S3 API and bucket policy design · object locking and versioning · data lifecycle management · storage replication reasoning · restore testing methodology
