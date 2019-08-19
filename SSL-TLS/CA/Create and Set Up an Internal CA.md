# Set Up an Internal CA (OpenSSL)

This procedure aims to provide a concise yet thorough coverage of the self-signed CA procedure.  Often development and testing require that SSL be enabled on their development and testing environments in order to emulate the production environments where their code will eventually run.  Commercially available certificates incur monetary costs, and as a result, self-signed certificates are advantageous in the development and testing environments.

#### 1. Create Directory to store CA certs
```
mkdir SelfSignCA
chmod 700 SelfSignCA
cd SelfSignCA
```

#### 2. Create CA Trust key and CA Certificate
```
openssl req -new -x509 -keyout ca.key -out ca.cert -days 3650
```

This key and certificate will be used to sign all server keys used on development and test environments

This openssl command will create a Self-Signed Signing CA certificate file called `ca.cert` and an associated private key file called `ca.key`. 
The certificate will have an expiration date 10 years (3650 days) into the future.  Keep these files in this secure directory as they are the basis of the trust structure for your group.  This command will require you to provide a password to protect the key.

eg:
```
# openssl req -new -x509 -keyout ca.key -out ca.cert -days 3650
Generating a 2048 bit RSA private key
...................................+++
...........................................................................................+++
writing new private key to 'ca.key'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:IN
State or Province Name (full name) []:Maharashtra
Locality Name (eg, city) [Default City]:Mumbai
Organization Name (eg, company) [Default Company Ltd]:BigData CA
Organizational Unit Name (eg, section) []:Development and Testing
Common Name (eg, your name or your server's hostname) []:RootCA
Email Address []:someemail@somecompany.com
#
# ll
total 20
-rw-r--r-- 1 root root 1493 Aug 01 19:53 ca.cert
-rw-r--r-- 1 root root 1834 Aug 01 19:53 ca.key
```

#### 3. Acting as the client requesting a signed certificate, create your key and keystore for each node in your cluster

Follow --> [Generate Cerificate Signing Request](https://github.com/dabsterindia/LABs/blob/master/SSL-TLS/CA/Obtain%20a%20Certificate%20from%20CA.md "Generate Cerificate Signing Request")


#### 4. Acting as the Signing Authority for your organization, create a signed certificate using the client CSR file:

Files needed:
 host.csr (created in step 3)
 ca.key (created in step 1)
 ca.cert (created in step 1)

Files created:
 host.crt

The command syntax used to create the certificate using the CSR file is:
```
openssl x509 -req -CA ca.cert -CAkey ca.key -in <host>.csr -out <host>.crt -days 730 -CAcreateserial -passin pass:<password>
```

eg:
```
# openssl x509 -req -CA ca.cert -CAkey ca.key -in master01.csr -out master01.crt -days 730 -CAcreateserial -passin pass:hadoop
Signature ok
subject=/C=IN/ST=Maharashtra/L=Pune/O=Dabster, Inc/OU=Development/CN=dabsterinc.com
Getting CA Private Key
```

#### 5. Send this file to Client with import instructions

#### 6. Acting as the client : import the certificate signed by Signing Authority above into your keystore

Follow --> [Import Certificate in Keystore](https://github.com/dabsterindia/LABs/blob/master/SSL-TLS/CA/Client%20-%20Import%20certificate%20signed%20by%20CA.md "Import Certificate in Keystore")
