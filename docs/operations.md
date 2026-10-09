
# Operations and rollback

## Health checks

```bash
kubectl get deployment,pod,svc,pvc -n media
kubectl describe deployment navidrome -n media
kubectl logs -n media deployment/navidrome --tail=100
```

Confirm the Deployment is ready, the Pod is Running, and both PVCs
are Bound. Confirm playback and album artwork through the application.

## Storage safety

The music source must remain read-only at every layer:

- PV: `spec.nfs.readOnly: true`
- NFS mount options include `ro`
- Pod volume mount: `readOnly: true`

Never remove the `ro` options as a troubleshooting shortcut.

## Rollback to the previous Docker deployment

The previous Docker container is retained until the migration is
accepted and recovery has been reviewed.

Before switching back:

1. Stop or scale down the Kubernetes Deployment to prevent two
   application instances from writing to separate or shared state.
2. Verify that the old Docker container's data directory and
   configuration are intact.
3. Start the old Docker deployment using its original configuration.
4. Verify login, playback, and album artwork.
5. Check logs and confirm the old application is healthy.

Do not delete the old Docker container or its data as part of routine
Kubernetes deployment. Do not assume that the Kubernetes database
and the original Docker database are synchronized after migration.

## Recovery limitations

The `local-path` PVC is not a backup. Before declaring the migration
complete, define an independent backup for `/data`, test restoration,
and document how to recover if node `media` or its storage is lost.
