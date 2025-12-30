Once GitOps is installed, you can create the app-of-apps for the hub cluster to bootstrap the cluster infrastructure and apply the ApplicationSets located in the dedicated path. 
For example, one of these ApplicationSets will install GitOps on the child clusters. 
To do so, simply run the following command:
```
oc apply -f provider-aws/apps/01-bootstrap/02-hub-infra/install
```
