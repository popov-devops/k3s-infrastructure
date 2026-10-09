
# Architecture

## Navidrome

Navidrome runs as a single-replica Kubernetes Deployment in namespace
`media`. A node selector pins the workload to node `media`.

The `Recreate` strategy avoids overlapping application instances that
might concurrently access the same SQLite database.

## Storage

| Component | Backend | Access | Purpose |
|---|---|---|---|
| `navidrome-data` | K3s `local-path` | Read/write | Database, config, application state |
| `navidrome-music` | OpenMediaVault NFSv3 | Read-only | Music library |

The NFS PV uses `Retain` reclaim policy. The NFS server export is
`/export/NAS4`; the Pod mounts its `jellyfin/music` subdirectory.

**OpenMediaVault is a read-only source for this workload.** Do not use
this deployment to write to or modify the source library.

The application PVC is node-local. If node `media` fails, recovery
requires restoring application data or recovering the node's local
storage. The repository contains configuration, not a backup of the
database or music.

## Networking

The Service exposes TCP port `4533` and NodePort `30453`.
Remote access currently uses an SSH tunnel rather than relying on
direct NodePort connectivity from outside the LAN.

The documented IP addresses are specific to the current homelab.
