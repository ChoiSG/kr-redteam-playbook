# LDAP Anonymous Bind





```
cme ldap <dc> -u '' -p '' --users 
cme ldap <dc> -u '' -p '' --groups
cme ldap <dc> -u '' -p '' --password-not-required
```





### 대응 방안

* ADSI Edit > "Configuration" > CN=Services > CN=Windows NT > CN=Directory Service > Properties > dSHeuristics = \<not set> OR `NOT 0000002`.&#x20;

<figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* Remove `Anonymous LOGON` groups having `READ` permission on `Users` Container.&#x20;

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>
