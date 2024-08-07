## Install the Helm Chart
```
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
for secret in secrets.${clusterName}/Secret-*.yaml;do name=$(echo ${secret}|cut -d/ -f2);kubeseal -f ${secret} -w templates/Sealed${name};done
```
