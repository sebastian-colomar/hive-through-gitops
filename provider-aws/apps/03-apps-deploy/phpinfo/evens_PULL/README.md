To deploy the PHPINFO sample application using the **PULL** model on the even-numbered child clusters (2 and 4), run the following command on the ACM hub cluster. 
This will create the ApplicationSet as well as the necessary placement:
```
oc apply -f provider-aws/apps/03-apps-deploy/phpinfo/evens_PULL/install
```
