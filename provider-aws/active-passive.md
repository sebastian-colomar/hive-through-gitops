# Configuring active-passive hub cluster

## Procedure

### On both the active and passive hub clusters:

1. You must install the Red Hat Advanced Cluster Management operator in the same namespace on both clusters
2. If the active hub cluster has any other operators installed, such as Ansible Automation Platform, Red Hat OpenShift GitOps, cert-manager, you have to install them also on the passive cluster.
   This ensures that the passive hub cluster is configured in the same way as the active hub cluster.
1. Edit the `MultiClusterHub` resource by setting the `cluster-backup` parameter to `true`.
   You can either use the graphical console for this task or run the following command line:
   ```
   oc patch multiclusterhub multiclusterhub -n ${NAMESPACE} --type='merge' -p '{"spec":{"overrides":{"components":[{"name":"cluster-backup","enabled":true}]}}}'
   ```
   This will install the OADP operator in the `open-cluster-management-backup` namespace.
   Wait until the installation is complete.
1. Create the storage location secret in the OADP Operator namespace, which is located in the backup component namespace:
   ```
   oc create secret generic cloud-credentials -n open-cluster-management-backup --from-file cloud=.aws/credentials
   ```
1. Create the instance of the `DataProtectionApplication` resource:
   ```
   oc create -f DataProtectionApplication.yaml
   ```   

### On the active hub cluster:

1. Schedule a cluster backup:
   ```
   oc create -f BackupSchedule.yaml
   ```

### On the passive hub cluster:

1. Use the `restore-acm-sync` sample to restore passive data, while continuing to check if new backups are available and restore them automatically.
   To automatically restore new backups, you must set the `syncRestoreWithNewBackups` parameter to `true`.
   You must also only restore the latest passive data.
   ```
   oc create -f restore-acm-sync.yaml
   ```

# Manual failover to the passive cluster

## Procedure

### On the active hub cluster:

1. To avoid getting into a backup collision state, delete the `BackupSchedule` resource on the active hub cluster before restoring managed clusters data on the passive hub cluster:
   ```
   oc delete -f BackupSchedule.yaml
   ```

### On the passive hub cluster:

1. Set the `VeleroManagedClustersBackupName` parameter to `latest`:
   ```
   oc patch restore restore-acm-sync -n open-cluster-management-backup --type=merge -p '{"spec": {"veleroManagedClustersBackupName": "latest"}}'
   ```
   Immediately after the `VeleroManagedClustersBackupName` is set to `latest`, the managed clusters are activated on the passive hub cluster and is now the primary hub cluster.
   When the activated managed cluster becomes the primary hub cluster, the `restore` resource is set to Finished and the `syncRestoreWithNewBackups` is ignored, even if set to `true`.
   Wait until the `restore` resource is finished.
1. Schedule a cluster backup storing the backups at the same storage location as the initial primary hub cluster: # IS IT ACTUALLY NECESSARY TO USE THE SAME LOCATION AS THE ORIGINAL PRIMARY HUB???
   ```
   oc create -f BackupSchedule.yaml
   ```
1. After these operations the passive hub cluster becomes the new active hub cluster.

### On the original active hub cluster:

1. Use the `restore-acm-sync` sample to restore passive data, while continuing to check if new backups are available and restore them automatically.
   To automatically restore new backups, you must set the `syncRestoreWithNewBackups` parameter to `true`.
   You must also only restore the latest passive data.
   ```
   oc create -f restore-acm-sync.yaml
   ```

# TODO:
## Connecting clusters automatically by using a Managed Service Account 

   
