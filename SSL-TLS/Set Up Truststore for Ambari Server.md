# Setup Truststore for Ambari Server


You must configure the Truststore for Ambari and add required certificates if:
  1) You have configured SSL for Ambari 
  2) You are using LDAPS to sync users from AD
  3) You are using SSL for HDP services


## Step 1. Create truststore file (On the Ambari Server)

```
cd /etc/ambari-server/conf/

keytool -import -file <path_to_the_SSL_Certificate> -alias ambari-server-crt -keystore ambari-server-truststore.jks

```

## Step 2. Configure the ambari-server to use this new trust store

`ambari-server setup-security`

```
Security setup options...
===========================================================================
Choose one of the following options:
  [1] Enable HTTPS for Ambari server.
  [2] Encrypt passwords stored in ambari.properties file.
  [3] Setup Ambari kerberos JAAS configuration.
  [4] Setup truststore.
  [5] Import certificate to truststore.
===========================================================================
Enter choice, (1-5): 4
Do you want to configure a truststore [y/n] (y)? y
TrustStore type [jks/jceks/pkcs12] (jks): jks
Path to TrustStore file : /etc/ambari-server/conf/ambari-server-truststore.jks
Password for TrustStore:
Re-enter password:
Ambari Server 'setup-security' completed successfully.
```

## Step 3. (Optinal) Verify truststore entries in ambari server configs (ambari.properties)

`grep -i 'truststore' /etc/ambari-server/conf/ambari.properties`

__Do you see any security Vulnerability here???__

Yes!!! the password of your truststore is configured in plain text.


Execute `ambari-server setup-security` and select option [2] to Encrypt passwords stored in ambari.properties file: [Ref](https://docs.hortonworks.com/HDPDocuments/Ambari-2.6.2.2/bk_ambari-security/content/optional_encrypt_database_and_ldap_passwords.html)



## Step 4. Restart ambari server

`ambari-server restart`
