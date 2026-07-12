# Operations

## Operational Model

This system is operated by a single technical contributor alongside other IT responsibilities. That constraint shaped every operational decision: if a procedure can't be executed reliably solo, under time pressure, it needs documentation before it's needed.

All recurring procedures are in the runbook. All incident responses follow a written process. Nothing critical depends on the operator remembering how to do it.

## Deployment

Initial cluster deployment follows a repeatable sequence:

1. Install Ubuntu Server LTS on each node — minimal install, static MAC-based DHCP reservation, SSH key auth only, password auth disabled
2. Install MicroK8s via snap on each node
3. Form the cluster — join worker nodes to the control plane via `microk8s add-node` / `microk8s join`
4. Enable required add-ons — `dns`, `helm3`, `rbac`, `metrics-server`
5. Deploy Longhorn via Helm — configure replica count, node tags, and recurring snapshot schedule
6. Deploy MinIO via Helm — configure tenant, bucket layout, access keys, object locking, and lifecycle policies
7. Configure switch VLANs and port assignments
8. Validate — confirm Longhorn volume health, MinIO bucket accessibility from each authorized client, and Kubernetes node status

Each step is documented with the exact commands run and expected output. A failed deploy can be diagnosed against the documented expected state.

## Monitoring

| What | How | Alert condition |
|---|---|---|
| Node health | `microk8s kubectl get nodes` + metrics-server | NotReady for > 5 min |
| Longhorn volume health | Longhorn UI / API | Volume degraded (replica below threshold) |
| Disk pressure | Node disk usage via metrics-server | > 80% utilization on any node |
| MinIO bucket usage | MinIO console metrics | Approaching capacity threshold |
| Backup recency | MinIO object timestamps per bucket | No new objects in expected window |

Monitoring checks are run on a defined schedule. Critical alerts (node down, volume degraded) are addressed immediately.

## Incident Response

### Node failure

1. Confirm node status — `microk8s kubectl get nodes`, `microk8s kubectl describe node <name>`
2. Check Longhorn — confirm volumes are degraded but not failed (replica count dropped, not zero)
3. Diagnose root cause on failed node — power, hardware, OS, disk
4. If node is recoverable: restore OS, rejoin cluster, confirm Longhorn rebuilds replicas
5. If node is not recoverable: provision replacement, rejoin, confirm replica rebuild

Data is safe as long as at least one replica remains healthy and is not lost before the failed node is restored or replaced.

### Volume degraded (Longhorn)

1. Longhorn UI — identify which volume is degraded and which replica is missing
2. Identify the node that lost the replica
3. If node is healthy: Longhorn will auto-rebuild — confirm rebuild completes
4. If node is not healthy: follow node failure procedure above

### Backup agent not writing

1. Check MinIO — confirm the bucket for the affected client exists and is healthy
2. Check access key — confirm the key is valid and scoped correctly to the bucket
3. Check ingestion VLAN — confirm client machine can reach the MinIO NodePort endpoint
4. Check agent-side logs on the client machine

## Maintenance

| Task | Frequency |
|---|---|
| Ubuntu security patches | Weekly (unattended-upgrades for non-kernel; manual for kernel) |
| MicroK8s version review | Quarterly |
| Longhorn snapshot cleanup | Automated via recurring job |
| Restore spot-test | Monthly — pull a recent object from a backup bucket and verify integrity |
| Switch config export | After any configuration change |
| Full cluster backup | Longhorn snapshots + MinIO object listing exported to offsite |

## Skills This Layer Built

Production runbook authorship · incident response procedure design · Kubernetes node lifecycle management · Longhorn operational procedures · monitoring design for a small cluster · maintenance scheduling for a sole contributor · restore testing methodology
