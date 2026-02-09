# DISCLAIMER
The following material is reproduced from the referenced guides and is provided exclusively for educational and training purposes.
It is provided on an "as-is" basis, without any express or implied warranties, and no responsibility is assumed for its accuracy, completeness, or applicability to any particular use.

---
# hive-through-gitops
## How to create an OCP cluster using Hive and ACM through GitOps (ArgoCD)

There are currently two folders with complete instructions on how to deploy for two different platform providers: [AWS](provider-aws) and [vSphere](provider-vsphere). I might add more examples in the future.

I'm also working on creating a [Helm](helm) chart to make implementation easier. But that's not ready right now.

The procedure is very simple:
1. There is an ArgoCD application with the name of the cluster that needs to be deployed to your GitOps platform (you must first install the GitOps operator in ACM and configure it to be able to connect to your Git repository, adding the necessary credentials). Here is an example of such an application: [Application-CLUSTER_NAME.yaml](provider-vsphere/Application-ocp-1.yaml)
1. There is a folder that has the name of the cluster that contains all the objects needed to create the OCP cluster. For example: [CLUSTER_NAME](provider-vsphere/ocp-1). Basically you will need the following:
    - [Hive cluster deployment](provider-vsphere/ClusterDeployment.yaml)
    - [RBAC cluster role](provider-vsphere/ClusterRole.yaml)
    - [RBAC cluster role binding](provider-vsphere/ClusterRoleBinding.yaml)
    - [ACM klusterlet addon config](provider-vsphere/KlusterletAddonConfig.yaml)
    - [Hive machine config pool](provider-vsphere/MachineConfigPool-worker.yaml) (for the worker nodes)
    - (optional) [Hive machine config pool](provider-vsphere/MachineConfigPool-infra.yaml) (for the infra nodes)
    - [ACM managed cluster](provider-vsphere/ManagedCluster.yaml)
    - [ACM managed cluster info](provider-vsphere/ManagedClusterInfo.yaml)
    - [RBAC role](provider-vsphere/Role.yaml)
    - [RBAC role binding](provider-vsphere/RoleBinding.yaml)
    - [Secret for the vSphere certificates](provider-vsphere/Secret-certs.yaml)
    - [Secret for the vSphere credentials](provider-vsphere/Secret-creds.yaml)
    - [Secret for the install config](provider-vsphere/Secret-install-config.yaml)
    - [Secret for the pull secret](provider-vsphere/Secret-pull-secret.yaml)
    - [Secret for the SSH private key](provider-vsphere/Secret-ssh-private-key.yaml)
1. You will need to customize the above folder with your own values. The most important part is the [RBAC cluster role](provider-vsphere/ClusterRole.yaml) - it is essential for the whole process to be successful. The secrets will be exposed in the Git repository, so it's a good idea to use [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) instead. Another option is to use managed policies to import the values of a previously created secret in the ACM hub cluster using [template functions](https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.7/html/governance/governance#fromsecret-func).
1. Once you create the ArgoCD application, GitOps will create the cluster. You will need to sync several times until the cluster is ready. If you [automate synchronization](https://argo-cd.readthedocs.io/en/stable/user-guide/auto_sync/), it is important that you disable it after the creation is successful.
1. The procedure to remove the cluster is as follows:
    1. First, [remove the cluster from the ACM cluster dashboard](https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.7/html/clusters/cluster_mce_overview#remove-managed-cluster).
    1. Once the cluster removal is successful, you can [remove the ArgoCD application](https://argo-cd.readthedocs.io/en/stable/user-guide/app_deletion/).
