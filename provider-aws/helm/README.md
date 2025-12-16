# How to create an OCP cluster on AWS using ACM, GitOps, and Helm

## Procedure

Install Red Hat Advanced Cluster Management for Kubernetes and create a MultiClusterHub instance.

Install Red Hat OpenShift GitOps.

Connect to the openshift-gitops-server URL of your ACM Hub cluster:
```
HUB_NAME=hub
DOMAIN=sebastian-colomar.com
https://openshift-gitops-server-openshift-gitops.apps.${HUB_NAME}.${DOMAIN}/settings/repos
```

Create a fork of this repository and connect it to GitOps:
```
USER=sebastian-colomar
REPO=hive-through-gitops
https://github.com/${USER}/${REPO}
```

Clone this git repository using a terminal with cluster-admin access to the Hub cluster and write access to Github:
```
git clone https://github.com/${USER}/${REPO}
```

Change directory into the repository folder:
```
cd ${REPO}
```

Choose the method and the provider:
```
method=helm
provider=aws
```

Create a new folder for the cluster settings files:
```
location=provider-${provider}/${method}
mkdir -p ${location}/${clusterName}
```

Choose the name of your cluster:
```
clusterId=1
clusterName=provider-${provider}-${method}-${clusterId}
```

Create the install-config.yaml:
```
vi ${location}/${clusterName}/install-config.yaml
```

Push the changes to the git repository:
```
git add ${location}/${clusterName}/install-config.yaml
git commit -m ${location}/${clusterName}/install-config.yaml
git push
```

Create the values.yaml:
```
vi ${location}/${clusterName}/values.yaml
```

Push the changes to the git repository:
```
git add ${location}/${clusterName}/values.yaml
git commit -m ${location}/${clusterName}/values.yaml
git push
```

Create the ApplicationSet.yaml:
```
vi ${location}/apps/ApplicationSet.yaml
```

Push the changes to the git repository:
```
git add ${location}/apps/ApplicationSet.yaml
git commit -m ${location}/apps/ApplicationSet.yaml
git push
```

Create a new project with the cluster name and create the necessary secrets for the installation configuration, platform credentials, Red Hat credentials and SSH private key:
```
clusterName=provider-${provider}-${method}-${clusterId}

oc new-project ${clusterName}

secretSuffix=install-config
oc create secret generic ${clusterName}-${secretSuffix} --from-file=${secretSuffix}.yaml=${location}/${clusterName}/${secretSuffix}.yaml --namespace ${clusterName}
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

Repeat the previous procedure for all cluster names:
```
clusterId=2
```
```
clusterId=3
```
```
clusterId=4
```

Once all the clusters have been preconfigured, apply the ApplicationSet that will deploy the clusters:
```
oc apply -f ${location}/ApplicationSet.yaml
```

To prevent unintended changes in your cluster, you can comment out the cluster element from the ApplicationSet after the cluster has been successfully created:
```
sed -i '/generators:/,/syncPolicy:/ {/- cluster: '\'"${clusterId}"\''/s/- cluster:/#- cluster:/}' ${location}/ApplicationSet.yaml
git add ${location}/ApplicationSet.yaml
git commit -m "Remove cluster ${clusterId} from ApplicationSet"
git push
```

You can destroy the cluster from the graphical dashboard:
- https://console-openshift-console.apps.openshift.sebastian-colomar.es/multicloud/infrastructure/clusters/managed

Alternatively, you can use the following command:
```
oc delete managedcluster ${clusterName}
oc delete clusterdeployment ${clusterName} --namespace=${clusterName}
```
