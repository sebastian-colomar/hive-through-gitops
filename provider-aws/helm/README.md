# How to create a new OCP cluster on AWS using ACM, GitOps, and Helm

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

provider=aws

location=provider-${provider}/${method}

clusterName=provider-${provider}-${method}-${clusterId}
```
Modify the install-config.yaml file as needed:
```
touch ${location}/install-config.${clusterName}.yaml

vi ${location}/install-config.${clusterName}.yaml
```
Push the changes to the git repository:
```
git add ${location}/install-config.${clusterName}.yaml

git commit -m ${location}/install-config.${clusterName}.yaml

git push
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
touch ${location}/ApplicationSet.yaml

vi ${location}/ApplicationSet.yaml
```
Push the changes to the git repository:
```
git add ${location}/ApplicationSet.yaml

git commit -m ${location}/ApplicationSet.yaml

git push
```
Create a new project with the cluster name and create the necessary secrets for the installation configuration, platform credentials, Red Hat credentials and SSH private key:
```
oc new-project ${clusterName}

secretSuffix=install-config
oc create secret generic ${clusterName}-${secretSuffix} --from-file=${secretSuffix}.yaml=${location}/${secretSuffix}.${clusterName}.yaml --namespace ${clusterName}
oc label secret ${clusterName}-${secretSuffix} --namespace=${clusterName} cluster.open-cluster-management.io/backup=cluster

secretName=aws-creds
secretNamespace=kube-system
secretSuffix=aws-creds
oc create secret generic ${clusterName}-${secretSuffix} --namespace ${clusterName}
for key in aws_access_key_id aws_secret_access_key;do
  oc patch secret ${clusterName}-${secretSuffix} --namespace=${clusterName} --patch='{"data":{"'${key}'":"'$(oc get secret ${secretName} --namespace=${secretNamespace} -ojsonpath="{.data.${key}}")'"}}'
done
oc label secret ${clusterName}-${secretSuffix} --namespace=${clusterName} cluster.open-cluster-management.io/backup=cluster cluster.open-cluster-management.io/copiedFromNamespace=${secretNamespace} cluster.open-cluster-management.io/copiedFromSecretName=${secretName}

secretName=pull-secret
secretNamespace=openshift-config
secretSuffix=pull-secret
oc create secret generic ${clusterName}-${secretSuffix} --namespace ${clusterName}
key=.dockerconfigjson
oc patch secret ${clusterName}-${secretSuffix} --namespace=${clusterName} --patch='{"data":{"'${key}'":"'$(oc get secret ${secretName} --namespace=${secretNamespace} -o json | jq -r '.data["'$key'"]')'"}}'
oc label secret ${clusterName}-${secretSuffix} --namespace=${clusterName} cluster.open-cluster-management.io/backup=cluster cluster.open-cluster-management.io/copiedFromNamespace=${secretNamespace} cluster.open-cluster-management.io/copiedFromSecretName=${secretName}

secretSuffix=ssh-private-key
oc create secret generic ${clusterName}-${secretSuffix} --namespace ${clusterName}
key=ssh-privatekey
oc patch secret ${clusterName}-${secretSuffix} --namespace=${clusterName} --patch='{"data":{"'${key}'":"'$(cat ${HOME}/.ssh/id_rsa | base64 | tr -d '\n')'"}}'
oc label secret ${clusterName}-${secretSuffix} --namespace=${clusterName} cluster.open-cluster-management.io/backup=cluster
```
Apply the ApplicationSet:
```
oc apply -f ${location}/ApplicationSet.yaml
```
To prevent unintended changes in your cluster, you can comment out the cluster element from the ApplicationSet after the cluster has been successfully created:
```
sed -i '/generators:/,/syncPolicy:/ {/- cluster: '\'"${clusterId}"\''/s/- cluster:/#- cluster:/}' ${location}/ApplicationSet.yaml

git add ${location}/ApplicationSet.yaml

git commit -m "Remove cluster ${clusterId} from ApplicationSet"

git push

oc apply -f ${location}/ApplicationSet.yaml
```
