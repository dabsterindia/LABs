# Setup and Test Knox

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
Output result will be displayed in JASON format. You can use any [JSON Formatter](https://jsonformatter.org/ "JSON Formatter") online, to format this json
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
### Setup AD/LDAP Authentication for knox
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
_Use below [Sample configuration](https://drive.google.com/open?id=1fgLWZt1gCWtgktt5Z9laaV4_FTDy1jl0 "Sample configuration Dabsterinc.com") to configure AD with Knox. You can refer websites given at the end of this
document to know more about these properties._
```
<?xml version="1.0" encoding="UTF-8"?>
<topology>
   <gateway>
      <provider>
         <role>authentication</role>
         <name>ShiroProvider</name>
         <enabled>true</enabled>
         <param>
            <name>sessionTimeout</name>
            <value>30</value>
         </param>
         <param>
            <name>main.ldapRealm</name>
            <value>org.apache.hadoop.gateway.shirorealm.KnoxLdapRealm</value>
         </param>
         <!-- changes for AD/user sync -->
         <param>
            <name>main.ldapContextFactory</name>
            <value>org.apache.hadoop.gateway.shirorealm.KnoxLdapContextFactory</value>
         </param>
         <param>
            <name>main.ldapRealm.contextFactory</name>
            <value>$ldapContextFactory</value>
         </param>
         <param>
            <name>main.ldapRealm.contextFactory.url</name>
            <value>ldap://adserver.dabsterinc.com:389</value>
         </param>
         <param>
            <name>main.ldapRealm.contextFactory.systemUsername</name>
            <value>CN=hdadmin,OU=hadoop,DC=dabsterinc,DC=lab,DC=com</value>
         </param>
         <param>
            <name>main.ldapRealm.contextFactory.systemPassword</name>
            <value>Hadoop@1</value>
         </param>
         <param>
            <name>main.ldapRealm.contextFactory.authenticationMechanism</name>
            <value>simple</value>
         </param>
         <!--
			<param>
				<name>main.ldapRealm.userDnTemplate</name>
				<value>CN={0},OU=DabUsers,OU=hadoop,DC=dabsterinc,DC=lab,DC=com</value>
			</param>
			-->
         <param>
            <name>main.ldapRealm.searchBase</name>
            <value>OU=DabUsers,OU=hadoop,DC=dabsterinc,DC=lab,DC=com</value>
         </param>
         <param>
            <name>main.ldapRealm.userObjectClass</name>
            <value>person</value>
         </param>
         <param>
            <name>main.ldapRealm.userSearchAttributeName</name>
            <value>sAMAccountName</value>
         </param>
         <param>
            <name>main.ldapRealm.authorizationEnabled</name>
            <value>true</value>
         </param>
         <param>
            <name>main.ldapRealm.groupSearchBase</name>
            <value>OU=DabUsers,OU=hadoop,DC=dabsterinc,DC=lab,DC=com</value>
         </param>
         <param>
            <name>main.ldapRealm.groupObjectClass</name>
            <value>group</value>
         </param>
         <param>
            <name>main.ldapRealm.groupIdAttribute</name>
            <value>cn</value>
         </param>
         <param>
            <name>urls./**</name>
            <value>authcBasic</value>
         </param>
      </provider>
      <provider>
         <role>authorization</role>
         <name>AclsAuthz</name>
         <enabled>true</enabled>
         <param name="knox.acl" value="*;*;*" />
      </provider>
      <provider>
         <role>identity-assertion</role>
         <name>Default</name>
         <enabled>true</enabled>
      </provider>
   </gateway>
   <!-- Services -->
   <service>
      <role>NAMENODE</role>
      <url>hdfs://{{namenode_host}}:{{namenode_rpc_port}}</url>
   </service>
   <service>
      <role>JOBTRACKER</role>
      <url>rpc://{{rm_host}}:{{jt_rpc_port}}</url>
   </service>
   <service>
      <role>WEBHDFS</role>
      {{webhdfs_service_urls}}
   </service>
   <service>
      <role>WEBHCAT</role>
      <url>http://{{webhcat_server_host}}:{{templeton_port}}/templeton</url>
   </service>
   <service>
      <role>OOZIE</role>
      <url>http://{{oozie_server_host}}:{{oozie_server_port}}/oozie</url>
   </service>
   <service>
      <role>WEBHBASE</role>
      <url>http://{{hbase_master_host}}:{{hbase_master_port}}</url>
   </service>
   <service>
      <role>HIVE</role>
      <url>http://{{hive_server_host}}:{{hive_http_port}}/{{hive_http_path}}</url>
   </service>
   <service>
      <role>RESOURCEMANAGER</role>
      <url>http://{{rm_host}}:{{rm_port}}/ws</url>
   </service>
   <service>
      <role>HDFSUI</role>
      <url>http://your.namenode.hostname.com:50070</url>
   </service>
   <service>
      <role>YARNUI</role>
      <url>http://your.rm.hostname.com:8088</url>
   </service>
   <service>
      <role>JOBHISTORYUI</role>
      <url>http://your.jhs.hostname.com:19888</url>
   </service>
   <service>
      <role>AMBARIUI</role>
      <url>http://your.ambari.hostname.com:8080</url>
   </service>
   <service>
      <role>RANGERUI</role>
      <url>http://your.ranger.hostname.com:6180</url>
   </service>
   <service>
      <role>DRUID-COORDINATOR-UI</role>
      {{druid_coordinator_urls}}
   </service>
   <service>
      <role>DRUID-COORDINATOR</role>
      {{druid_coordinator_urls}}
   </service>
   <service>
      <role>DRUID-OVERLORD-UI</role>
      {{druid_overlord_urls}}
   </service>
   <service>
      <role>DRUID-OVERLORD</role>
      {{druid_overlord_urls}}
   </service>
   <service>
      <role>DRUID-ROUTER</role>
      {{druid_router_urls}}
   </service>
   <service>
      <role>DRUID-BROKER</role>
      {{druid_broker_urls}}
   </service>
   <service>
      <role>ZEPPELINUI</role>
      {{zeppelin_ui_urls}}
   </service>
   <service>
      <role>ZEPPELINWS</role>
      {{zeppelin_ws_urls}}
   </service>
</topology>
```

__Note: (For Ranger Authorization)__
To enable Ranger for Knox, find ‘AclsAuthz’ string in the above topology and replace with
‘XASecurePDPKnox’.

### Validate Above configs:

#### 1) Make sure Knox is configured to use CA certificates
```
# openssl s_client -showcerts -connect knoxhostname.dabster.com:8443
```
#### 2) Validate Topology definition
```
# /usr/hdp/current/knox-server/bin/knoxcli.sh validate-topology --cluster default
File to be validated:
/usr/hdp/2.6.5.0-292/knox/bin/../conf/topologies/default.xml
==========================================
Topology file validated successfully
```

#### 3) Test LDAP Authentication and Authorization
```
# /usr/hdp/current/knox-server/bin/knoxcli.sh user-auth-test [--cluster c] [--u username] [
--p password] [--g] [--d]
```

```
[root@hdp1 deployments]# /usr/hdp/current/knox-server/bin/knoxcli.sh user-auth-test --cluster HortonProd --u dab01
--p Hadoop@1

org.apache.hadoop.gateway.util.KnoxCLI$LDAPCommand$NoSuchTopologyException: Topology Hort
onProd does not exist in the topologies directory.
[root@hdp1 deployments]# cd /etc/knox/conf/topologies/
[root@hdp1 topologies]# ll
total 24
-rw-r--r-- 1 knox knox 1820 Aug 14 16:36 admin.xml
-rw-r--r-- 1 knox knox 3176 Aug 16 16:52 default.xml
-rw-r--r-- 1 knox knox 3307 Aug 14 16:36 knoxsso.xml
-rw-r--r-- 1 knox knox 4968 May 11 07:51 manager.xml
-rw-r--r-- 1 knox knox 89 May 11 07:51 README

[root@hdp1 topologies]# /usr/hdp/current/knox-server/bin/knoxcli.sh user-auth-test --clus
ter default --u dab01 --p Hadoop@1
LDAP authentication successful!
```

#### 4) Test the ability to connect, bind, and authenticate with the LDAP server
```
[root@hdp1 topologies]# /usr/hdp/current/knox-server/bin/knoxcli.sh system-user-auth-test --cluster default --d
System LDAP Bind successful.
```

#### 5) List Topologies
```
[root@hdp1 topologies]# /usr/hdp/current/knox-server/bin/knoxcli.sh list-topologies
```
### Access WebHDFS with & without KNOX Using AD/lDAP credentials

#### 1) Without KNOX
```
# curl http://node2:50070/webhdfs/v1/?op=LISTSTATUS
```

#### 2) With KNOX
```
# curl -k -u hadoopadmin:Hadoop@1 "https://knox.dabsterinc.com:8443/gateway/default/webhdfs/v1/?op=LISTSTATUS"
```

```
# curl -k -u hr1:Hadoop@1 "https://knox.dabsterinc.com:8443/gateway/default/webhdfs/v1/user?op=LISTSTATUS"
```

```
# curl -k -u dab01:Hadoop@1 "https://knox.dabsterinc.com:8443/gateway/default/webhdfs/v1/?op=LISTSTATUS"
```

#### 3) You can also use WebBrowser to execute below Knox URL.
Open Chrome and past Knox URL you want to execute. Ex:
https://knox.dabsterinc.com:8443/gateway/default/webhdfs/v1/?op=LISTSTATUS"

