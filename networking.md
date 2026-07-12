# Networking

## Design Goals

The network has three requirements that shaped every decision:

1. **Isolation** — storage/data-plane traffic, cluster management, and client backup ingestion should not share a broadcast domain.
2. **Local-only access** — no service is reachable from outside the physical premises. This is a deliberate compliance posture, not a limitation.
3. **Simplicity** — a sole technical contributor needs to be able to reason about the full network state without tooling assistance.

## Physical Topology

A single managed switch connects all cluster nodes. VLANs partition the switch into logical segments. Nodes are trunked to carry relevant VLANs; access ports drop untagged traffic into the appropriate segment.

```
                        [Managed Switch]
                        /      |      \
               [Control]  [Worker 1]  [Worker 2]
                  trunk      trunk      trunk
```

## VLAN Segmentation

| Segment | Traffic type | Who accesses it |
|---|---|---|
| Management | SSH, kubectl, Longhorn UI, switch management | Authorized operators only |
| Data plane | Longhorn inter-node replication, etcd | Cluster nodes only |
| Ingestion | MinIO S3 endpoint (backup agent traffic) | Authorized client machines |

Separating Longhorn replication traffic onto its own VLAN was necessary for two reasons: replication is bandwidth-intensive and would otherwise compete with management traffic, and it keeps storage internal state off any segment a backup agent could reach.

## Switch Configuration

The switch was configured via CLI (not web UI) — the same workflow used on production managed switches. Key configuration areas:

- VLAN database and naming
- Trunk ports for nodes carrying multiple VLANs
- Access port assignment for single-VLAN endpoints
- Port security — MAC-based restrictions on access ports
- STP (Spanning Tree Protocol) port configuration to prevent loops

All configuration is documented in a local runbook. Switch config is exported and version-controlled separately.

## Local-Access Model

No services are exposed beyond the local network. This was a deliberate architecture choice, not a constraint.

Rationale:
- Client data never transits the public internet (ingestion happens on-premises or over a secured internal connection)
- Eliminates an entire class of external threat surface
- Simplifies the compliance posture — no public-facing endpoints to audit

Remote administration (for the sole maintainer) uses an on-premises workstation on the management VLAN. No VPN, no tunnel, no external dependency for day-to-day operations.

## Skills This Layer Built

Managed switch configuration via CLI · VLAN database design · trunk and access port configuration · port security · traffic segmentation reasoning · network documentation
