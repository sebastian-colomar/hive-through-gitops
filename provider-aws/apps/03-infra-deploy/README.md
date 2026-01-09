Once GitOps is installed on the hub cluster and the child managed clusters have been deployed, you can create the app-of-apps for the hub cluster to bootstrap the cluster infrastructure that will apply the ApplicationSets located in the designated path (for example, one of these ApplicationSets will install GitOps on the child clusters). 

To create the app-of-apps for the hub cluster, simply run the following command:
```
oc apply -f provider-aws/apps/03-infra-deploy/bootstrap
```
