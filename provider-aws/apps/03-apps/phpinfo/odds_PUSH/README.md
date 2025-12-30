To deploy the PHPINFO sample application using the **PUSH** model on the odd-numbered child clusters (1 and 3), run the following command on the ACM hub cluster. 
This will create the ApplicationSet as well as the necessary placement:
```
oc apply -f provider-aws/apps/03-apps/phpinfo/odds_PULL/install
```
