# Import the certificates in keystore

#### 1 . Get cetificate signed by CA
Follow --> [This](https://github.com/dabsterindia/LABs/blob/master/SSL-TLS/CA/Create%20and%20Set%20Up%20an%20Internal%20CA.md "") to generate new certificates, if you dont have CA signed certs.

#### 2. Import the certificate signed by Signing Authority above into your keystore
Files needed:
 ca.cert (This is RootCA certificate)
 host.crt (Your host certificate)

#### 3. Import the trusted CA certificate `ca.cert` in the the keystore
```
keytool -keystore keystore.jks -alias CARoot -import -file ca.cert
```

eg:
```
# keytool -import -keystore keystore.jks -alias RootCA -file ca.cert
Enter keystore password:
Owner: EMAILADDRESS=someemail@somecompany.com, CN=RootCA, OU=Development and Testing, O=BigData CA, L=Mumbai, ST=Maharashtra, C=IN
Issuer: EMAILADDRESS=someemail@somecompany.com, CN=RootCA, OU=Development and Testing, O=BigData CA, L=Mumbai, ST=Maharashtra, C=IN
Serial number: ad038ffbe15974b4
Valid from: Mon Aug 19 19:53:29 UTC 2019 until: Thu Aug 16 19:53:29 UTC 2029
Certificate fingerprints:
	 MD5:  B5:B8:32:6C:95:C6:2D:B6:32:99:D8:EA:88:8C:95:CF
	 SHA1: C0:1E:15:93:D0:B1:F2:EB:9E:B8:E7:80:6E:60:21:C5:D0:F1:4F:4D
	 SHA256: D6:FE:02:BD:97:5F:82:24:98:DD:59:A3:40:28:13:91:FF:9C:E8:3E:63:43:80:0E:B5:3A:14:7B:0B:C2:4C:F5
	 Signature algorithm name: SHA256withRSA
	 Version: 3

Extensions:

#1: ObjectId: 2.5.29.35 Criticality=false
AuthorityKeyIdentifier [
KeyIdentifier [
0000: 64 21 2E 37 48 AD AF 83   A7 A8 35 A2 A6 1B BB A7  d!.7H.....5.....
0010: F3 3F 5F 54                                        .?_T
]
]

#2: ObjectId: 2.5.29.19 Criticality=false
BasicConstraints:[
  CA:true
  PathLen:2147483647
]

#3: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: 64 21 2E 37 48 AD AF 83   A7 A8 35 A2 A6 1B BB A7  d!.7H.....5.....
0010: F3 3F 5F 54                                        .?_T
]
]

Trust this certificate? [no]:  yes
Certificate was added to keystore
```

#### 4. Import server certificate
```
keytool -keystore keystore.jks -alias <host> -import -file <host>.crt
```

#### 5. Confirm that the CA and signed certificate are in your keystore
```
keytool -list -v -keystore keystore.jks
```
