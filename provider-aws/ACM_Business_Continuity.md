# ACM Business Continuity

## Procedure

### Configuring active-passive hub cluster 

#### On both the active and passive hub clusters:

1. You must install the Red Hat Advanced Cluster Management operator in the same namespace on both clusters
2. If the active hub cluster has any other operators installed, such as Ansible Automation Platform, Red Hat OpenShift GitOps, cert-manager, you have to install them also on the passive cluster.
   This ensures that the passive hub cluster is configured in the same way as the active hub cluster.
1. Edit the MultiClusterHub resource by setting the cluster-backup parameter to true.
   You can either use the graphical console for this task or run the following command line:
   ```
   oc patch multiclusterhub multiclusterhub -n ${NAMESPACE} --type='merge' -p '{"spec":{"overrides":{"components":[{"name":"cluster-backup","enabled":true}]}}}'
   ```
   This will install the OADP operator in the open-cluster-management-backup namespace.
   Wait until the installation is complete.
1. Create the storage location secret in the OADP Operator namespace, which is located in the backup component namespace:
   ```
   oc create secret generic cloud-credentials -n open-cluster-management-backup --from-file cloud=.aws/credentials
   ```
1. Create the instance of the DataProtectionApplication resource:
   ```
   oc create -f DataProtectionApplication.yaml
   ```   
