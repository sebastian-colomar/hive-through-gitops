1. Edit the MultiClusterHub resource by setting the cluster-backup parameter to true

   This will install the OADP operator in the open-cluster-management-backup namespace.
   Wait until the installation is complete.
   
1. Create a storage location secret:
   ```
   oc create secret generic cloud-credentials -n open-cluster-management-backup --from-file cloud=.aws/credentials
   ```
1. Create an instance of the DataProtectionApplication resource:
   ```
   oc create -f DataProtectionApplication.yaml
   ```
1. Schedule a cluster backup:
   ```
   oc create -f BackupSchedule.yaml
   ```
