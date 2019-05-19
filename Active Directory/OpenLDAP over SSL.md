# Enable SSL/TLS Security for OpenLDAP server



### 1. Create SSL certificates
```
[root@dabsterinc ~]# cd /etc/pki/tls/certs 
```

#### 1.1. Create private key
```
[root@dabsterinc certs]# make server.key 
```

#### 1.2. Remove passphrase from private key
```
[root@dabsterinc certs]# openssl rsa -in server.key -out server.key
Enter pass phrase for server.key:
writing RSA key
```

#### 1.3. Create Certificate Signing Request and provide details as per your environment setup
```
[root@dabsterinc certs]# make server.csr
umask 77 ; \
/usr/bin/openssl req -utf8 -new -key server.key -out server.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:IN
State or Province Name (full name) []:Maharashtra
Locality Name (eg, city) [Default City]:Pune
Organization Name (eg, company) [Default Company Ltd]:Dabster,Inc
Organizational Unit Name (eg, section) []:IT
Common Name (eg, your name or your server's hostname) []:myldap.dabsterinc.com
Email Address []:support@dabsterinc.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

#### 1.4. Create Certificate
```
[root@dabsterinc certs]# openssl x509 -in server.csr -out server.crt -req -signkey server.key -days 3650
Signature ok
subject=/C=IN/ST=Maharashtra/L=Pune/O=Dabster,Inc/OU=IT/CN=c1230-node1.squadron-labs.com
Getting Private key
```
#### 1.5. Move certificate and key to openldap certs directory
```
[root@dabsterinc ~]# cp /etc/pki/tls/certs/server.key \
/etc/pki/tls/certs/server.crt \
/etc/pki/tls/certs/ca-bundle.crt \
/etc/openldap/certs/ 
```
#### 1.6. Make sure files are owened by `ldap` user
```
[root@dabsterinc ~]# chown ldap. /etc/openldap/certs/server.key \
/etc/openldap/certs/server.crt \
/etc/openldap/certs/ca-bundle.crt
```

### 2. Configure LDAP server to use SSL certificate

#### 2.1. Create a file and provide following details
```
[root@dabsterinc ~]# vi mod_ssl.ldif
```

```
dn: cn=config
changetype: modify
add: olcTLSCACertificateFile
olcTLSCACertificateFile: /etc/openldap/certs/ca-bundle.crt
-
replace: olcTLSCertificateFile
olcTLSCertificateFile: /etc/openldap/certs/server.crt
-
replace: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/openldap/certs/server.key
```

#### 2.2. Push the config
```
[root@dabsterinc ~]# ldapmodify -Y EXTERNAL -H ldapi:/// -f mod_ssl.ldif 
```

#### 2.3. Edit `slapd` config file and add `ldaps:///` at line# 9
```
[root@dlp ~]# vi /etc/sysconfig/slapd
```
```
# line 9: add
SLAPD_URLS="ldapi:/// ldap:/// ldaps:///"
```

### 3. Restart OpenLDAP server
```
[root@dabsterinc ~]# systemctl restart slapd 
```

### 4. Configure LDAP Client for TLS connection.
```
[root@dabsterinc ~]# echo "TLS_REQCERT allow" >> /etc/openldap/ldap.conf
[root@dabsterinc ~]# echo "tls_reqcert allow" >> /etc/nslcd.conf
[root@dabsterinc ~]# authconfig --enableldaptls --update
```

### 5. Run ldapsearch and verify certificate
```
[root@dabsterinc ~]# ldapsearch -x -H ldaps://localhost:636 -D 'cn=Manager,dc=dabsterinc,dc=com' -w 'dabster123!' -b 'ou=Hadoop,dc=dabsterinc,dc=com' uid=john
```
