# Challenges

## Storage for Regulated Client Data

**Status:** Solved

**Problem:** Client backup data carries retention and protection requirements. A naive solution — a shared NFS export, or a single large disk — would make retention enforcement, per-client isolation, and point-in-time recovery either impossible or manual. Manual compliance procedures are a liability.

**Solution:** Split storage by access pattern and requirement. Longhorn handles durability (replication across nodes, snapshot scheduling). MinIO handles isolation and retention enforcement (per-client buckets, object locking, lifecycle policies). Each layer does what it's best at. Compliance properties are configured once, in code, and enforced automatically — not by operator discipline.

**Skills:** storage architecture, compliance-aware design, Kubernetes storage primitives

---

## VLAN Segmentation on a Managed Switch

**Status:** Solved

**Problem:** Three distinct traffic types — Longhorn inter-node replication, cluster management, and client backup ingestion — were sharing the same untagged network. Longhorn replication saturated available bandwidth during heavy writes, which degraded management responsiveness. More importantly, backup ingestion agents should never reach the management or storage-internal segments.

**Solution:** Configured VLANs via CLI on the managed switch. Assigned trunk ports to nodes (carrying management + data-plane VLANs), access ports to single-purpose endpoints. Separated ingestion traffic onto its own segment — agents can reach the MinIO endpoint and nothing else.

**Skills:** managed switch CLI configuration, VLAN trunk/access port design, traffic segmentation, network security posture

---

## S3-Compatible Backup Ingestion Without Cloud

**Status:** Solved

**Problem:** Most backup agents support S3 as a target. The alternatives (NFS, SMB, SFTP) are narrower, less consistently implemented, and harder to enforce access controls on. But cloud S3 introduces egress costs, external data transfer, and compliance complexity for regulated data.

**Solution:** MinIO deployed on-cluster. Agents write to a local S3 endpoint — same API calls, same bucket semantics, zero external dependency. Changing the underlying storage implementation later doesn't require touching the agents.

**Skills:** MinIO deployment and administration, S3 API, on-premises object storage architecture

---

## Heterogeneous Node Hardware

**Status:** Managed

**Problem:** The cluster nodes are repurposed desktop machines with differing CPU generations, RAM, and disk capacity. Kubernetes doesn't know that, and naive scheduling can place storage workloads on underpowered or low-capacity nodes.

**Solution:** Longhorn node tagging. Nodes are tagged by storage capacity tier. Longhorn scheduling rules prevent replica placement on nodes below threshold. The cluster is aware of its own hardware heterogeneity and schedules accordingly.

**Skills:** Kubernetes node tagging, Longhorn scheduling configuration, heterogeneous hardware management

---

## Sole Contributor Operational Scope

**Status:** Managed

**Problem:** A system handling regulated data, operated by one person alongside other responsibilities, has no margin for undocumented procedures. If the operator is unavailable or simply forgets a step under pressure, data integrity or availability could be affected.

**Solution:** Every procedure is in the runbook before it's needed. Deployment is a numbered sequence with expected output at each step. Incident response is a written decision tree. Recurring maintenance is scheduled and tracked. The system is designed to be operated by one person because every step is written down.

**Skills:** technical documentation, runbook authorship, operational design for small teams

---

## MicroK8s Certificate Expiry

**Status:** Solved

**Problem:** MicroK8s internal certificates have a finite validity period. On a long-running cluster with infrequent disruption, this can catch an operator off-guard — the cluster stops accepting API calls with a confusing TLS error.

**Solution:** Diagnosed via `microk8s inspect` and certificate validity checks. Resolved with `microk8s refresh-certs`. Added certificate expiry to the maintenance calendar.

**Skills:** Kubernetes TLS certificate management, MicroK8s lifecycle operations, root-cause analysis under operational pressure
