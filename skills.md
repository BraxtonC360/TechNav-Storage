# Skills

Skills built operating production infrastructure in a professional context. This system handles real client data with real operational consequences — retention requirements, data integrity obligations, and availability expectations are not optional. That context is different from a lab or course project, and it shaped how each skill below was learned.

I'm a CS/Math student (Maryville College, junior). These skills were built alongside coursework, not in place of it.

---

## Kubernetes & Platform Engineering

Learned by operating a production MicroK8s cluster as sole maintainer.

**Cluster lifecycle management**
- Forming a multi-node cluster from scratch — joining workers to the control plane, verifying quorum, confirming etcd health
- Enabling and configuring add-ons (`dns`, `rbac`, `helm3`, `metrics-server`)
- Diagnosing and recovering from node failures, including `NotReady` states and API server connectivity issues
- Refreshing expired internal certificates (`microk8s refresh-certs`) — learned under operational pressure, not from a tutorial

**Workload management**
- Deploying stateful workloads via Helm — values file configuration, upgrade procedures, rollback
- PersistentVolumeClaim design — StorageClass selection, access modes, capacity planning
- Namespace and RBAC design — ServiceAccounts, Roles, RoleBindings scoped by workload
- Configuring resource requests and limits to prevent workloads from starving each other on heterogeneous hardware

**Operational visibility**
- `kubectl` proficiency — not just `get` and `describe`; event inspection, log following, exec into pods for live debugging
- metrics-server for node-level resource visibility
- Reading Longhorn's health model — understanding what degraded vs. faulted means and what action each state requires

*Relevant to: platform engineering, infrastructure engineering, SRE, any role running Kubernetes in production*

---

## Distributed Storage

Learned by designing and operating the storage layer for a compliance-sensitive backup system.

**Longhorn**
- Deploying and configuring Longhorn on a production cluster
- Replica count configuration and the trade-off between redundancy and write amplification
- Node tagging to control replica placement across heterogeneous hardware
- Volume snapshot scheduling — recurring jobs, retention, and manual snapshot procedures
- Diagnosing degraded volumes and monitoring rebuild progress
- Understanding the difference between a degraded volume (recoverable) and a faulted volume (data at risk)

**Storage architecture reasoning**
- Separating workloads by access pattern — block storage for persistent state, object storage for ingest
- Understanding replication as a durability mechanism, not a backup mechanism
- Capacity planning on constrained hardware: how much space does replication actually consume, and how does that change with replica count

*Relevant to: storage engineering, systems engineering, iXsystems/TrueNAS-adjacent roles, any infrastructure role with a storage component*

---

## Object Storage & S3

Learned by deploying MinIO as the backup ingestion endpoint for a production system.

**MinIO administration**
- Deploying MinIO on Kubernetes via Helm
- Tenant and bucket layout design — per-source bucket isolation, naming conventions
- Access key management — generating scoped keys, assigning bucket-level permissions, rotating credentials
- Object locking (WORM) — understanding lock modes (Compliance vs. Governance), setting retention periods, verifying lock enforcement
- Versioning — enabling versioning, understanding delete marker behavior, recovering from accidental deletion
- Lifecycle policies — configuring automatic expiration rules for retention compliance

**S3 API**
- How backup agents authenticate and write to S3-compatible endpoints
- Bucket policy JSON — what it controls and how it differs from IAM-style access keys
- Using the MinIO client (`mc`) for administration, data migration, and spot-checks

*Relevant to: storage engineering, cloud infrastructure, platform engineering, any role touching object storage or backup systems*

---

## Networking

Learned by designing and configuring the network for a production cluster — on real managed switch hardware, via CLI.

**Managed switch configuration**
- VLAN database setup and naming
- Trunk port configuration — carrying multiple VLANs to cluster nodes
- Access port assignment — single-VLAN endpoints, untagged traffic handling
- Port security — MAC-based restrictions to prevent unauthorized devices from joining segments
- STP port configuration — preventing loops on a small switched network
- Configuration export and documentation

**Network architecture reasoning**
- VLAN design by traffic type — separating storage replication, cluster management, and client ingestion
- Understanding why traffic segmentation is both a performance and a security decision
- Local-access network architecture — when not exposing services externally is the right call, and how to structure access accordingly

*Relevant to: network engineering, infrastructure engineering, finance-infra roles (where network segmentation and controlled access are non-negotiable)*

---

## Linux System Administration

Learned by provisioning and maintaining production Ubuntu Server nodes.

- Ubuntu Server LTS installation and initial hardening — SSH key auth, disabled password auth, unattended security updates
- Static DHCP reservation and hostname management across a multi-node cluster
- Service management via `systemd` — status, restart, journal inspection for MicroK8s and cluster services
- Disk management — partition layout for cluster nodes, understanding the impact of disk I/O on storage workload performance
- Network interface configuration for VLAN trunk ports on server nodes

*Relevant to: any infrastructure or systems engineering role; baseline expectation for most of the roles I'm targeting*

---

## Compliance-Aware Infrastructure Design

Learned by operating a system that handles client data under applicable retention and data protection requirements.

This isn't a skill you get from a textbook. It's learned by asking the question "what happens if this goes wrong?" for every design decision, and then actually implementing the answer.

Key concepts applied in practice:
- **Data isolation** — no shared storage between clients; a misconfiguration for one client cannot affect another's data
- **Retention enforcement** — WORM locking and lifecycle policies enforce retention at the storage layer, not by operator discipline
- **Auditability** — every procedure is documented; every change to the system can be traced
- **Minimize exposure** — local-only access, no external endpoints, smallest possible access scope for every credential

*Relevant to: finance-infra roles (where regulatory compliance is a first-class concern), any role touching regulated or sensitive data*

---

## Production Ownership as a Sole Contributor

Possibly the hardest skill to learn from a course, and the one most relevant to a small-team or early-stage infrastructure role.

Operating this system solo means:
- Every design decision is mine — there's no senior engineer to catch a bad call before it hits production
- Every incident is mine — no rotation, no on-call handoff, no escalation path
- Every procedure must be written down — not because it's required, but because memory is not a reliable operational dependency

Skills built:
- Writing runbooks that are actually useful under pressure (not just documentation theater)
- Designing monitoring that surfaces real problems rather than noise
- Making architecture decisions with operational burden as a first-class input — a clever solution that's hard to operate solo is a bad solution

*Relevant to: any early-career infrastructure role; especially relevant to finance-infra and startup-adjacent positions where engineers own their systems end-to-end*
