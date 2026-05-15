# Velero Disaster Recovery Runbook

## Restore a Namespace

1. List available backups:
```bash
   velero backup get
```

2. Restore from backup:
```bash
   velero restore create --from-backup <backup-name> --wait
```

3. Verify restore:
```bash
   velero restore get
   kubectl get all -n <namespace>
```

## Restore Entire Cluster

1. Install Velero on new cluster
2. Point to same MinIO bucket
3. List backups: `velero backup get`
4. Restore: `velero restore create --from-backup full-backup-<date>`

## Backup Location

- MinIO Server: http://10.0.0.52:9001
- Storage: steven LXC (ID 200)
- Bucket: velero
- Schedule: Daily at 2 AM
- Retention: 30 days

## Recovery Time Objective (RTO)

Tested: ~3 minutes for namespace restore
