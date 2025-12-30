Once GitOps is installed on the hub cluster and the child managed clusters have been deployed, you can create the app-of-apps for the hub cluster to bootstrap the cluster infrastructure and apply the ApplicationSets located in the designated path.
For example, one of these ApplicationSets will install GitOps on the child clusters. 
To do so, simply run the following command:
```
oc apply -f provider-aws/apps/03-infra-deploy/bootstrap
```
