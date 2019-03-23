# Set Up SSL for Ambari


## Step 1. Log into to Ambari Server node

`ssh ambari.server.hostname`


## Step 2. Create a directory to store certificates
```
mkdir /etc/ambari-server/conf/certs
cd /etc/ambari-server/conf/certs/
```

## Step 3. Create a VARIABLE (wserver) to host ambari-server host fqdn

```
export wserver=$(hostname -f)
echo $wserver
```

## Step 4. For CA signed certificates, skip this step and start from Step 5

For Self-Signed Certificates follow these steps to generate new certificate:
If you want to create a temporary self-signed certificate, use this as an example:

#### 4.1 : Generate a new private key and Certificate Signing Request
```
cd /var/tmp/
openssl genrsa -out $wserver.key 2048
openssl req -new -key $wserver.key -out $wserver.csr
```

```
-----
Country Name (2 letter code) [XX]:IN
State or Province Name (full name) []:Maharashtra
Locality Name (eg, city) [Default City]:Pune
Organization Name (eg, company) [Default Company Ltd]:Dabster, Inc
Organizational Unit Name (eg, section) []:Hadoop Support
Common Name (eg, your name or your server's hostname) []:master1.hadoop.com
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```


#### 4.2 : Generate a self-signed certificate
```
openssl x509 -req -days 365 -in $wserver.csr -signkey $wserver.key -out $wserver.crt
```

## Step 5. Move SSL certificate & Private key to a directory created in step 2

```
cp /var/tmp/$wserver.crt /etc/ambari-server/conf/certs/
cp /var/tmp/$wserver.key /etc/ambari-server/conf/certs/
ls -lrt /etc/ambari-server/conf/certs/
```

## Step 6. (Optional) verify the 
If you need to check the information within a Certificate, CSR or Private Key, use these commands.

`openssl x509 -in /etc/ambari-server/conf/certs/$wserver.crt -text -noout`


## Step 7. Setup SSL for ambari-server

#### 7.1 : Stop Ambari server

`ambari-server stop`

#### 7.2 : Run setup-security command

`ambari-server setup-security`


```
Using python  /usr/bin/python
Security setup options...
===========================================================================
Choose one of the following options:
  [1] Enable HTTPS for Ambari server.
  [2] Encrypt passwords stored in ambari.properties file.
  [3] Setup Ambari kerberos JAAS configuration.
  [4] Setup truststore.
  [5] Import certificate to truststore
===========================================================================
Enter choice, (1-5): 1
Do you want to configure HTTPS [y/n] (y)? y
SSL port [8443] ?
Enter path to Certificate: /etc/ambari-server/conf/certs/master1.hadoop.com.crt
Enter path to Private Key: /etc/ambari-server/conf/certs/master1.hadoop.com.key
Please enter password for Private Key:
Importing and saving Certificate...done.
Ambari server URL changed. To make use of the Tez View in Ambari please update the property tez.tez-ui.history-url.base in tez-site
Adjusting ambari-server permissions and ownership...
```

#### 7.2 : Start ambari server
`ambari-server start`


## Step 8. Login Ambari server over https protocol & 8443 port


![alt text](https://github.com/dabsterindia/LABs/blob/master/SSL-TLS/ambari-server%20SSL%20UI-%20Self%20Signed.png)
      

## Step 9. Configure Ambari truststore

Follow ---------- article to setup ambari truststore
