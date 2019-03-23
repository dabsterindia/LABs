# Set Up SSL for Ambari


References:
https://www.sslshopper.com/article-most-common-openssl-commands.html
https://www.sslshopper.com/article-most-common-java-keytool-keystore-commands.html

https://support.ssl.com/Knowledgebase/Article/View/19/0/der-vs-crt-vs-cer-vs-pem-certificates-and-how-to-convert-them


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

## Step 4. For CA signed certificates, Skip this step. You will have to get the certificate and store it in a directory we
created in step 2 and start from Step 5.

For Self-Signed Certificates follow these steps to generate new certificate:
If you want to create a temporary self-signed certificate, use this as an example:

#### 4.1 : Generate a new private key and Certificate Signing Request
```
cd /var/tmp/
openssl genrsa -out $wserver.key 2048
openssl req -new -key $wserver.key -out $wserver.csr
```

#### 4.2 : Generate a self-signed certificate
```
openssl x509 -req -days 365 -in $wserver.csr -signkey $wserver.key -out $wserver.crt
```

Step 5. Move SSL certificate & Private key to a directory created in step 2

```
cp /var/tmp/$wserver.crt /etc/ambari-server/conf/certs/
cp /var/tmp/$wserver.key /etc/ambari-server/conf/certs/
```

Step 6. (Optional) verify the 
If you need to check the information within a Certificate, CSR or Private Key, use these commands.

`openssl x509 -in /etc/ambari-server/conf/certs/$wserver.crt -text -noout`


Step 7. Setup SSL for ambari-server

#### 7.1 : Stop Ambari server & run setup-security command
```
ambari-server stop

ambari-server setup-security
```

```



```








