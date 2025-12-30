# How to create an OCP cluster on AWS using ACM, GitOps, and Helm

## Procedure

Install Red Hat Advanced Cluster Management for Kubernetes and create a MultiClusterHub instance.

Install Red Hat OpenShift GitOps and connect this repository to GitOps.

Clone this git repository using a terminal with cluster-admin access to the Hub cluster and write access to Github:
```
git clone https://github.com/sebastian-colomar/hive-through-gitops
```

Change directory into the repository folder:
```
cd hive-through-gitops
```

If necessary, modify the current cluster configuration and settings, then commit the changes to the remote repository and proceed.
You may need to fork this repository in order to have write permissions.

Now you need to prepare the necessary resources before the actual cluster deployment.
Create a new project with the cluster name and create the necessary secrets for the installation configuration, platform credentials, Red Hat credentials and SSH private key:
```
for N in {1..4};do

clusterId=$N

method=helm
provider=aws
location=provider-${provider}/apps/01-minions-deployment/${method}

clusterName=provider-${provider}-${method}-${clusterId}

oc new-project ${clusterName}

secretSuffix=install-config
oc create secret generic ${clusterName}-${secretSuffix} --from-file=${secretSuffix}.yaml=${location}/clusters/${clusterName}/${secretSuffix}.yaml --namespace ${clusterName}
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

done
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

Alternatively, you can safely delete the ApplicationSet once all the clusters have been successfully created. 
This will prevent any interference with the existing clusters.

When you need to destroy the cluster, you can do it from the graphical dashboard:
- https://console-openshift-console.apps.openshift.sebastian-colomar.es/multicloud/infrastructure/clusters/managed

Alternatively, you can use the following command:
```
oc delete managedcluster ${clusterName}
oc delete clusterdeployment ${clusterName} --namespace=${clusterName}
```
