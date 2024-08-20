## Install the Helm Chart
```
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets

helm repo update

helm install sealed-secrets -n kube-system --set-string fullnameOverride=sealed-secrets-controller sealed-secrets/sealed-secrets
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
clusterName=xxx
```
```
for secret in secrets.${clusterName}/Secret-*.yaml;do name=$(echo ${secret}|cut -d/ -f2);kubeseal -f ${secret} -w templates/Sealed${name};done

key=aws_access_key_id
keyId=${key}
value=$(awk -F "${key}: " '{print $2}' templates/SealedSecret-*|grep .|sed 's/\//\\\//g');sed -i "s/${keyId}.*$/${keyId}: ${value}/" values.${clusterName}.yaml

key=aws_secret_access_key
keyId=${key}
value=$(awk -F "${key}: " '{print $2}' templates/SealedSecret-*|grep .|sed 's/\//\\\//g');sed -i "s/${keyId}.*$/${keyId}: ${value}/" values.${clusterName}.yaml

key=install-config.yaml
keyId=installConfig
value=$(awk -F "${key}: " '{print $2}' templates/SealedSecret-*|grep .|sed 's/\//\\\//g');sed -i "s/${keyId}.*$/${keyId}: ${value}/" values.${clusterName}.yaml

key=.dockerconfigjson
keyId=pullSecret
value=$(awk -F "${key}: " '{print $2}' templates/SealedSecret-*|grep .|sed 's/\//\\\//g');sed -i "s/${keyId}.*$/${keyId}: ${value}/" values.${clusterName}.yaml

key=ssh-privatekey
keyId=sshPrivateKey
value=$(awk -F "${key}: " '{print $2}' templates/SealedSecret-*|grep .|sed 's/\//\\\//g');sed -i "s/${keyId}.*$/${keyId}: ${value}/" values.${clusterName}.yaml
```
