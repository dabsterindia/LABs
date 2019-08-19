# Obtain a Certificate from a Trusted Third-Party Certification Authority (CA)

A third-party Certification Authority (CA) accepts certificate requests from entities, authenticates applications, issues certificates, and maintains status information about certificates. Associated cryptography guarantees that a signed certificate is computationally difficult to forge. Thus, as long as the CA is a genuine and trusted authority, clients have high assurance that they are connecting to the machines that they are attempting to connect with.

To obtain a certificate signed by a third-party CA, generate and submit a Certificate Signing Request (CSR) for each cluster node:


#### 1. Generate the host key:
```
keytool -keystore <client-keystore> -genkey -alias <host>
```
#### 2. At the prompts, enter the information required by the CSR.
```
[root@master01 ~]# keytool -keystore keystore.jks -genkey -alias `hostname`
Enter keystore password:
Re-enter new password:
What is your first and last name?
  [Unknown]:  master02.dabsterinc.com
What is the name of your organizational unit?
  [Unknown]:  Development
What is the name of your organization?
  [Unknown]:  Dabster, Inc
What is the name of your City or Locality?
  [Unknown]:  Pune
What is the name of your State or Province?
  [Unknown]:  Maharashtra
What is the two-letter country code for this unit?
  [Unknown]:  IN
Is CN=master02.dabsterinc.com, OU=Development, O="Dabster, Inc", L=Pune, ST=Maharashtra, C=IN correct?
  [no]:  yes

Enter key password for <master02.dabsterinc.com>
	(RETURN if same as keystore password):
Re-enter new password:
```
By default, `keystore` uses JKS format for the keystore and truststore. 

#### 3. Verify that the key was generated
 ```
 keytool -list -v -keystore keystore.jks
 ```

#### 4. Create the CSR file
```
keytool -keystore <keystorename> -certreq -alias <host> -keyalg rsa -file <host>.csr
```
This command generates a certificate signing request that can be sent to a CA. The file <host>.csr contains the CSR.

#### 5. Submit the CSR to your Certificate Authority.

#### 6. To import and install keys and certificates, follow the instructions sent to you by the CA

Ref: 
https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.3.4/bk_Security_Guide/content/ch_obtain-trusted-cert.html
