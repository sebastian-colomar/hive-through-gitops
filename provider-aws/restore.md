1. Destroy the original ACM hub cluster
2. Create a new OCP hub cluster
3. Install ACM
4. Install any other operator that was in the original hub cluster (like GitOps)
5. Wait until the installation is complete
1. Edit the MultiClusterHub resource by setting the cluster-backup parameter to true

   This will install the OADP operator in the open-cluster-management-backup namespace.
   Wait until the installation is complete.
1. Create the storage location secret:
   ```
   oc create secret generic cloud-credentials -n open-cluster-management-backup --from-file cloud=.aws/credentials
   ```
1. Create the instance of the DataProtectionApplication resource:
   ```
   oc create -f DataProtectionApplication.yaml
   ```
1. Restore the backup:
   ```
   oc create -f Restore.yaml
   ```
