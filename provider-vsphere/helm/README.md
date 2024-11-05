# How to create an OCP cluster on vSphere using ACM, GitOps, and Helm

## Procedure

Install Red Hat Advanced Cluster Management for Kubernetes and create a MultiClusterHub instance.

Install Red Hat OpenShift GitOps.

Connect to the openshift-gitops-server URL:
- https://openshift-gitops-server-openshift-gitops.apps.openshift.sebastian-colomar.es/settings/repos

Connect a new repository:
- https://github.com/sebastian-colomar/hive-through-gitops

Clone this git repository:
```
git clone https://github.com/sebastian-colomar/hive-through-gitops
```
Change directory into the repository folder:
```
cd hive-through-gitops/
```
Choose the name of your cluster:
```
clusterId=1
```
Launch the following commands:
```
method=helm

provider=vsphere

providerLocation=provider-${provider}/${method}

clusterName=provider-${provider}-${method}-${clusterId}

secretName=xxx

secretNamespace=openshift-config
```
Create a new folder for the cluster settings files:
```
location=${providerLocation}/${clusterName}

mkdir -p ${location}
```
Modify the install-config.yaml file as needed:
```
touch ${location}/install-config.${clusterName}.yaml

vi ${location}/install-config.${clusterName}.yaml
```
Modify the values.${clusterName}.yaml file as needed:
```
touch ${location}/values.${clusterName}.yaml

vi ${location}/values.${clusterName}.yaml
```
Push the changes to the git repository:
```
git add ${location}/values.${clusterName}.yaml

git commit -m ${location}/values.${clusterName}.yaml

git push
```
Modify the ApplicationSet.yaml file as needed:
```
touch ${providerLocation}/ApplicationSet.yaml

vi ${providerLocation}/ApplicationSet.yaml
```
Launch the script to create the cluster:
```
source ${providerLocation}/cluster-create.sh
```
This script will create a new project with the cluster name and create the necessary secrets for the installation configuration, platform credentials, Red Hat credentials and SSH private key. In the end, it will apply the ApplicationSet.

To prevent unintended changes in your cluster, you can comment out the cluster element from the ApplicationSet AFTER the cluster has been SUCCESSFULLY created:
```
sed -i '/generators:/,/syncPolicy:/ {/- cluster: '\'"${clusterId}"\''/s/- cluster:/#- cluster:/}' ${location}/ApplicationSet.yaml

oc apply -f ${location}/ApplicationSet.yaml
```
When you need to destroy the cluster, you can do it from the graphical dashboard:
- https://console-openshift-console.apps.openshift.sebastian-colomar.es/multicloud/infrastructure/clusters/managed

Alternatively, you can use the following command to DESTROY the cluster:
```
oc delete managedcluster ${clusterName}

oc delete clusterdeployment ${clusterName} -n ${clusterName}
```
