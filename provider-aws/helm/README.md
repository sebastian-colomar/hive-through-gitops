# How to create a new OCP cluster using ACM, GitOps and Helm

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
Launch the following commands:
```
clusterId=1

method=helm

provider=aws

location=provider-${provider}/${method}

clusterName=provider-${provider}-${method}-${clusterId}
```
Modify the install-config.yaml file as needed.
```
touch ${location}/install-config.${clusterName}.yaml

vi ${location}/install-config.${clusterName}.yaml
```
Modify the values.${clusterName}.yaml file as needed.
```
touch ${location}/values.${clusterName}.yaml

vi ${location}/values.${clusterName}.yaml
```
Create a new project with the cluster name and create the necessary secrets for the installation configuration, platform credentials, Red Hat credentials and SSH private key:
```
oc new-project ${clusterName}

secretName=aws-cloud-credentials
secretNamespace=openshift-machine-api

secretSuffix=install-config
oc create secret generic ${clusterName}-${secretSuffix} --from-file=${secretSuffix}.yaml=${location}/${secretSuffix}.${clusterName}.yaml --namespace ${clusterName}
oc label secret ${clusterName}-${secretSuffix} --namespace=${clusterName} cluster.open-cluster-management.io/backup=cluster

secretSuffix=aws-creds
oc create secret generic ${clusterName}-${secretSuffix} --namespace ${clusterName}
key=aws_access_key_id
keyId=${key}
oc patch secret ${clusterName}-${secretSuffix} --namespace=${clusterName} --patch='{"data":{"'${key}'":"'$(oc get secret ${secretName} --namespace=${secretNamespace} -ojsonpath="{.data.${keyId}}")'"}}'
key=aws_secret_access_key
keyId=${key}
oc patch secret ${clusterName}-${secretSuffix} --namespace=${clusterName} --patch='{"data":{"'${key}'":"'$(oc get secret ${secretName} --namespace=${secretNamespace} -ojsonpath="{.data.${keyId}}")'"}}'
oc label secret ${clusterName}-${secretSuffix} --namespace=${clusterName} cluster.open-cluster-management.io/backup=cluster cluster.open-cluster-management.io/copiedFromNamespace=${secretNamespace} cluster.open-cluster-management.io/copiedFromSecretName=${secretName}

secretSuffix=pull-secret
oc create secret generic ${clusterName}-${secretSuffix} --namespace ${clusterName}
key=.dockerconfigjson
keyId=pullSecret
oc patch secret ${clusterName}-${secretSuffix} --namespace=${clusterName} --patch='{"data":{"'${key}'":"'$(oc get secret ${secretName} --namespace=${secretNamespace} -ojsonpath="{.data.${keyId}}")'"}}'
oc label secret ${clusterName}-${secretSuffix} --namespace=${clusterName} cluster.open-cluster-management.io/backup=cluster cluster.open-cluster-management.io/copiedFromNamespace=${secretNamespace} cluster.open-cluster-management.io/copiedFromSecretName=${secretName}

secretSuffix=ssh-private-key
oc create secret generic ${clusterName}-${secretSuffix} --namespace ${clusterName}
key=ssh-privatekey
keyId=sshPrivateKey
oc patch secret ${clusterName}-${secretSuffix} --namespace=${clusterName} --patch='{"data":{"'${key}'":"'$(oc get secret ${secretName} --namespace=${secretNamespace} -ojsonpath="{.data.${key}}"  )'"}}'
oc label secret ${clusterName}-${secretSuffix} --namespace=${clusterName} cluster.open-cluster-management.io/backup=cluster cluster.open-cluster-management.io/copiedFromNamespace=${secretNamespace} cluster.open-cluster-management.io/copiedFromSecretName=${secretName}
```
Push the changes
```
git add ${location}/values.${clusterName}.yaml

git commit -m ${location}/values.${clusterName}.yaml

git push
```
