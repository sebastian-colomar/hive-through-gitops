## Install the Helm Chart
```
helm repo add bitnami https://charts.bitnami.com/bitnami

helm repo update

helm install sealed-secrets -n kube-system --set-string fullnameOverride=sealed-secrets-controller bitnami/sealed-secrets
```

## Fetch the latest sealed-secrets version using GitHub API
```
KUBESEAL_VERSION=$(curl -s https://api.github.com/repos/bitnami-labs/sealed-secrets/tags | jq -r '.[0].name' | cut -c 2-)

curl -OL "https://github.com/bitnami-labs/sealed-secrets/releases/download/v${KUBESEAL_VERSION}/kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz"

tar -xvzf kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz kubeseal

sudo cp kubeseal /usr/local/bin/
```

## Encrypt the secrets with Kubeseal
```
for secret in secrets.${clusterName}/Secret-*.yaml;do name=$(echo ${secret}|cut -d/ -f2);kubeseal -f ${secret} -w templates/Sealed${name};done
```

## Create the necessary role and role binding
```
oc create rolebinding openshift-gitops-argocd-application-controller --namespace=${clusterName} --role=openshift-gitops-argocd-application-controller --serviceaccount=openshift-gitops:openshift-gitops-argocd-application-controller

oc create role openshift-gitops-argocd-application-controller --namespace=${clusterName} --verb=create --resource=applications --api-group=argoproj.io
```
