# Install & Setup phpLDAPadmin on CentOS 7


### 1. Update system packages
```
# yum update && sudo yum upgrade -y
```
### 2. Install extra PHP packages
```
# yum install php-ldap php-mbstring php-pear php-xml -y
```
### 3. Install Extra Packages for Enterprise Linux (EPEL) release
```
# yum install epel-release
```


### 4. Install the phpLDAPadmin
```
# yum -y install phpldapadmin
```

### 5. Configure the phpLDAPadmin Virtual Host

#### a) Configure the phpLDAPadmin Virtual Host
```
# vi  /etc/httpd/conf.d/phpldapadmin.conf
```

```
Alias /phpldapadmin /usr/share/phpldapadmin/htdocs
Alias /ldapadmin /usr/share/phpldapadmin/htdocs

<Directory /usr/share/phpldapadmin/htdocs>
  <IfModule mod_authz_core.c>
    # Apache 2.4
    #Require local
    Require all granted
  </IfModule>
  <IfModule !mod_authz_core.c>
    # Apache 2.2
    Order Deny,Allow
    Deny from all
    Allow from 127.0.0.1
    Allow from ::1
  </IfModule>
</Directory>
```
#### b)Configure the phpLDAPadmin
```
# vi /etc/phpldapadmin/config.php
```
_uncomment line __298___
```
$servers->setValue('server','host','127.0.0.1');
```
_Line __332__ will define your domain details, change it appropriately._
```
$servers->setValue('login','bind_id','cn=ldapadmin,dc=hortonworks,dc=com');
```

_The password hashing algorithm set should be ssha. So change line __388__ appropriately:_
```
$servers->setValue('appearance','password_hash','ssha');
```

_Uncommented Line __397__ & Comment out line __398__
```
$servers->setValue('login','attr','dn');
//$servers->setValue('login','attr','uid');
```
_Save and Exit


### 6. Retart LDAP services
```
# systemctl restart slapd
```

### 7. Login phpldapadmin 

URL : http://(serverIP)/phpldapadmin



![alt text](https://github.com/dabsterindia/LABs/blob/master/tmp/images/phpldapadmin.png)
