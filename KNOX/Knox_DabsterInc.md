#Setup and Test Knox

### Validating Service Connectivity Using DEMO-LDAP (Optional)
After adding the knox service, we can validate whether know is working fine. Knox comes with the demoldap to perform this test.

#### Step 1: Start demo ldap from Ambari Dashboard

_Ambari > Knox > Service Action > Start DemoLDAP_


#### Step 2: Access WebHDFS using HDFS API(without Knox Gateway) and Using Knox Gateway

##### a) Using WebHDFS API
```
# curl -k "http://namenode.hadoop.com:50070/webhdfs/v1/user?op=LISTSTATUS"
```

##### b) Using Knox gateway
```
# curl -k -u guest:guest-password "https://knox.hadoop.com:8443/gateway/default/webhdfs/v1/user?op=LISTSTATUS"
```

In Above example, we have listed the directories under /user using Webhdfs API call and knox gateway.
Output result will be displayed in JASON format. You can use any [JSON formatter] (https://jsonformatter.org/ "Json Formatter") online, to format this json
output.

#### Step 3: Write file to HDFS using Knox gatway
##### a) Define Variable
```
# KNOX_HOSTNAME=knoxhost.dabsterinc.com
# USERNAME="op1"
# PASS="Hadoop@1"
```

##### b) Register the name for a sample file README in /tmp
```
# curl -i -k -u $USERNAME:$PASS -X PUT "https://$KNOX_HOSTNAME:8443/gateway/default/webhdfs/v1/tmp/README?op=CREATE"
```

##### c) Upload README to /tmp directory on HDFS (hint: Use Value of Location header from command above)
```
# curl -i -k -u $USERNAME:$PASS -X PUT '{https://hdp1.c.verdant-rider-212512.internal:8443/gateway/default/webhdfs/data/v1/webhdfs/v1/tmp/README?_=AAAACAAAABAAAADQ5LY8-eixe9bO-w8d4seSLyCaQzLCSFmavvXrNpySilHJzY0IO9pK3UPYwdWeZlxN650S4B7iiFNtKTyvryqyGTZNYvRg8DI16PQ0e3Zyzpp7nvEBN6F1X4P-98qoiUO3XG0OJpMR_iXr4R2hrLq_mK0r0PZnaDWw0mK7GX4a_KmQK_nVNB9j5OU6BTWKlo_jGf7mUPbcUwepYViFopCFDn56BF_DVPF5gw9FyVHtIaqgRhd9K1kuxIFzghwVtOayJfWMedCm49BwwWb0rYkbqsb-QlFWlnjem1egHNTYAH8IowcXrVzS_Q}'
```

##### d) Verify whether file is uploaded or not
```
[root@hdp1 2.2.0]# hadoop fs -ls /tmp
Found 3 items
-rwxr-xr-x 3 op1 hdfs 0 2018-08-16 16:36 /tmp/README
drwxr-xr-x - hdfs hdfs 0 2018-08-06 12:46 /tmp/entity-file-history
drwx-wx-wx - ambari-qa hdfs 0 2018-08-06 12:56 /tmp/hive
```

### Creating Knox topology
### Setup Up LDAP/AD Authentication for knox
LDAP authentication is configured by adding a "ShiroProvider" authentication provider to the cluster's topology
file. When enabled, the Knox Gateway uses Apache Shiro `(org.apache.shiro.realm.ldap.JndiLdapRealm or
org.apache.hadoop.gateway.shirorealm.KnoxLdapRealm)` to authenticate users against the configured LDAP store.

#### 1) Add proxy user details for Knox in core-site.xml
```
hadoop.proxyuser.knox.groups=*
hadoop.proxyuser.knox.hosts=*
```
#### 2) Configure Knox Topology for AD/LDAP Authentication (Make sure you Stop Demo LDAP if it is running)

#### 3) Goto : Ambari > Knox > Config > Advance Topology
_Use below [Sample configuration](https://drive.google.com/open?id=1fgLWZt1gCWtgktt5Z9laaV4_FTDy1jl0 "Dabsterinc.com Knox Sample") to configure AD with Knox. You can refer websites given at the end of this
document to know more about these properties._





