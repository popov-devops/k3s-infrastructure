
# K3s Homelab Infrastructure

Infrastructure-as-code repository for a self-hosted K3s cluster.
The project documents service deployment, storage, networking,
migration from Docker, and operational recovery procedures.

## Current workload

- **Navidrome** — self-hosted music streaming server.
- **Deployment:** one replica, pinned to the `media` node.
- **Container image:** `deluan/navidrome:0.63.2`.
- **Application data:** 1 GiB PVC using K3s `local-path`.
- **Music library:** NFSv3 volume backed by OpenMediaVault.
- **Service:** NodePort `30453`, service port `4533`.

## Storage design

Navidrome application state is stored separately from the music library.

The music library is an external NFS source and must remain read-only.
Read-only is enforced by the PV, NFS mount options, and container mount.

The `local-path` volume is node-local storage, not shared storage.
A PVC does not replace an independent backup.

## Access

The service is exposed as NodePort `30453`.
For remote access, the current working method is an SSH tunnel to a
cluster node that can reach the `media` node:

```bash
ssh -N -L 4533:192.168.88.157:30453 USER@SSH_HOST
```

Then open `http://127.0.0.1:4533` locally.
Replace `USER` and `SSH_HOST` with your actual SSH account and host.
Do not commit private keys, credentials, tokens, or kubeconfig files.

## Deployment

Review the manifests before applying them. The NFS server address is
specific to this homelab and must be adjusted for another environment.

```bash
kubectl create namespace media
kubectl apply -f media/navidrome/pv-music.yaml
kubectl apply -f media/navidrome/pvc-music.yaml
kubectl apply -f media/navidrome/pvc-data.yaml
kubectl apply -f media/navidrome/deployment.yaml
kubectl apply -f media/navidrome/service.yaml
```

If the `media` namespace already exists, `kubectl create namespace media`
may report that it already exists; this is harmless.

## Validation

```bash
kubectl get deployment,pod,service,pvc -n media
kubectl get pv navidrome-music
kubectl logs -n media deployment/navidrome
```

Check that the Deployment is ready and both PVCs are Bound.
Verify that the music volume is mounted read-only before relying on it.

## Documentation

- [Architecture](docs/architecture.md)
- [Operations and rollback](docs/operations.md)
