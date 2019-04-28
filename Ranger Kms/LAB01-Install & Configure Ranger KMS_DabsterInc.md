# Install and Configure Ranger KMS


## Pre-requisites (Optional)
If you do not have database admin credentials, create a database, db user and grant him required privileges.


#### 1) Install Mysql Server
```
# yum install -y mysql-server
# service mysqld start
# chkconfig mysqld on
```

#### 2) Create database
```
# mysql

mysql> create database rangerkms;
```

#### 3) Create db user & grant permissions
```
create user 'rangerkms'@'%' identified by 'rangerkms';
grant all privileges on rangerkms.* to 'rangerkms'@'%';

create user 'rangerkms'@'localhost' identified by 'rangerkms';
grant all privileges on rangerkms.* to 'rangerkms'@'localhost';

create user 'rangerkms'@'kms-master.dabsterinc.com' identified by 'rangerkms';
grant all privileges on rangerkms.* to 'rangerkms'@'kms-master.dabsterinc.com';

flush privileges;
```
## Add Ranger KMS service through Ambari UI
Go to Ambari Dashboard and add service

#### 1) Ambari > Add service > Ranger KMS
#### 2) At costomized Service section, Provide database and complete the installation

## Configure Ranger KMS

### 1) Configure KMS properties
##### Add/Verify below properties in Ranger KMS > Configs

##### - Advance-kms property
`REPOSITORY_CONFIG_USERNAME=keyadmin`

##### - Custom kms-site
```
hadoop.kms.proxyuser.hive.users=*
hadoop.kms.proxyuser.hive.hosts=*
hadoop.kms.proxyuser.oozie.users=*
hadoop.kms.proxyuser.oozie.hosts=*
hadoop.kms.proxyuser.ambari.users=*
hadoop.kms.proxyuser.ambari.hosts=*
hadoop.kms.proxyuser.yarn.users=*
hadoop.kms.proxyuser.yarn.hosts=*
hadoop.kms.proxyuser.HTTP.users=*
hadoop.kms.proxyuser.HTTP.hosts=*

hadoop.kms.proxyuser.keyadmin.groups=*
hadoop.kms.proxyuser.keyadmin.hosts=*
hadoop.kms.proxyuser.keyadmin.users=*
```
##### - Verify below in "advanced kms-site" (If kerberos enabled)
```
hadoop.kms.authentication.type=kerberos
hadoop.kms.authentication.kerberos.keytab=/etc/security/keytabs/spnego.service.keytab
hadoop.kms.authentication.kerberos.principal=*
```

##### - Enable Ranger KMS Audit. Goto Advanced ranger-kms-audit
`xasecure.audit.is.enabled = true`


### 2) Create softlink on RangerKMS node
```
ssh kms-master.dabsterinc.com

$ sudo ln -s /etc/hadoop/conf/core-site.xml /etc/ranger/kms/conf/core-site.xml
```

## Configure HDFS Encryption to use Ranger KMS Access
##### 1) Add/Verify below properties in Ambari > HDFS > Configs

##### - Verify "Advanced core-site"
```
hadoop.security.key.provider.path = kms://http@kms_hostname:9292/kms
dfs.encryption.key.provider.uri = kms://http@kms_hostname:9292/kms
```

##### - Add in Custom core-site.xml
`hadoop.proxyuser.kms.groups=*`

##### 2) (Optional) Create new hdfs admin user(superuser):
```
$ adduser kadmin
$ usermod -G hdfs kadmin
```

Verify:
```
# su - kadmin

# hdfs groups kadmin
kadmin : kadmin hdfs

$ hdfs dfsadmin -report
```

## Restart Required Services

## Follow Ranger KMS LAB TEST to test whether kms is functioning as expected
