# SSL Certificate Management SOP

> **Organization:** Bu9a Arabia  
> **Department:** IT Infrastructure  
> **Environment:** Production (Cloudera / CDH Cluster)  
> **Last Updated:** 2025

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Phase 1: Generate CSR (Certificate Signing Request)](#phase-1-generate-csr)
4. [Phase 2: Create & Distribute Certificates](#phase-2-create--distribute-certificates)
5. [Phase 3: Extract & Organize Certificates](#phase-3-extract--organize-certificates)
6. [Phase 4: Create Keystore (PKCS12 & JKS)](#phase-4-create-keystore)
7. [Host Reference](#host-reference)

---

## Overview

This SOP covers the end-to-end process for generating SSL Certificate Signing Requests (CSRs), importing signed certificates into the Java truststore, distributing CA chain files to cluster nodes, and creating keystores for the Cloudera CDH cluster at Bu9a Arabia.

---

## Prerequisites

- OpenSSL installed on the working machine
- Java 1.8 (OpenJDK) installed
- SSH access to all cluster nodes
- Signed certificates (`rootca.pem`, `intca.pem`) available from the CA
- `keytool` available via `$JAVA_HOME/bin`

---

## Phase 1: Generate CSR

### Step 1 — Create Working Directory

```bash
mkdir -p /var/tmp/SSL/CSR
```

---

### Step 2 — Create Host List File

Create a file listing all FQDNs for which CSRs need to be generated.

```bash
cat > /var/tmp/SSL/CSR/prodhosts.txt <<EOF
bjhqcdpmgr01.dabsterinc.com
EOF
```

> **Note:** Add one FQDN per line for multiple hosts.

---

### Step 3 — Create SAN Configuration Template

This template is used to generate per-host OpenSSL config files with Subject Alternative Name (SAN) support.

```bash
cat > /var/tmp/SSL/CSR/SAN.cnf <<EOF
[ req ]
default_bits       = 2048
distinguished_name = req_distinguished_name
req_extensions     = req_ext
prompt = no

[ req_distinguished_name ]
C = SA
ST = JED
L = Jeddah
O = Bu9a Arabia
OU = IT
CN = host_fqdn

[ req_ext ]
subjectAltName = @alt_names
extendedKeyUsage = serverAuth, clientAuth

[alt_names]
DNS.1   = host_fqdn
EOF
```

| Field | Value |
|-------|-------|
| Country | SA |
| State | JED |
| Locality | Jeddah |
| Organization | Bu9a Arabia |
| OU | IT |
| CN / SAN | FQDN of each host |

---

### Step 4 — Generate CSR and Private Key for Each Host

```bash
cd /var/tmp/SSL/CSR
mkdir tmp-sanconf

for fqdn in `cat prodhosts.txt`
do
  sed "s/host_fqdn/$fqdn/g" SAN.cnf > ./tmp-sanconf/$fqdn-san.cnf
  openssl req -out $fqdn.csr \
              -newkey rsa:2048 \
              -nodes \
              -keyout $fqdn.key \
              -config ./tmp-sanconf/$fqdn-san.cnf
done
```

This will produce:
- `<fqdn>.csr` — Certificate Signing Request (submit to CA)
- `<fqdn>.key` — Private key (keep secure, do not share)

---

### Step 5 — Archive and Submit CSRs

```bash
ls -lrt /var/tmp/SSL/CSR
tar -czvf prod-csr.tar.gz *.csr
```

> **Action:** Send `prod-csr.tar.gz` to the Certificate Authority (CA) for signing.

---

## Phase 2: Create & Distribute Certificates

### Step 6 — Set Java Environment

```bash
JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.292.b10-1.el7_9.x86_64
export PATH=$PATH:$JAVA_HOME/bin
```

---

### Step 7 — Merge CA Chain

Combine the Intermediate CA and Root CA into a single chain file.

```bash
cat intca.pem rootca.pem > cacerts-chain.pem
```

---

### Step 8 — Import CA Certificates into Java Truststore

```bash
# Import Root CA
keytool -import \
  -keystore $JAVA_HOME/jre/lib/security/cacerts \
  -file rootca.pem \
  -alias  bu9aArabiaRoot

# Import Intermediate CA
keytool -import \
  -keystore $JAVA_HOME/jre/lib/security/cacerts \
  -file intca.pem \
  -alias  bu9aArabiaSub1
```

> **Note:** Default Java truststore password is `changeit` unless changed by your team.

---

### Step 9 — Distribute CA Chain Files to All Cluster Nodes

```bash
for i in `cat ~/prodhosts`
do
  echo -e "------------------------\n $i \n------------------------\n"
  scp intca.pem rootca.pem cacerts-chain.pem $i:/opt/cloudera/security/pki/
done
```

> **Optional pre-steps** (uncomment in script if PKI directory does not exist on target hosts):
> ```bash
> ssh $i 'sudo mkdir -p /opt/cloudera/security/pki; chmod -R 755 /opt/cloudera/security'
> scp $JAVA_HOME/jre/lib/security/cacerts $i:$JAVA_HOME/jre/lib/security/cacerts
> ```

---

## Phase 3: Extract & Organize Certificates

### Step 10 — Extract Signed Certificates

Once signed `.pem` files are received back from the CA, extract only the certificate block.

```bash
mkdir certs
```

**Manager Nodes:**
```bash
cat bjhqcdpmgr01.pem | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > certs/bjhqcdpmgr01.dabsterinc.com.pem
cat bjhqcdpmgr02.pem | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > certs/bjhqcdpmgr02.dabsterinc.com.pem
```

**Data Nodes:**
```bash
cat bjhqcdp01.pem | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > certs/bjhqcdp01.dabsterinc.com.pem
cat bjhqcdp02.pem | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > certs/bjhqcdp02.dabsterinc.com.pem
cat bjhqcdp03.pem | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > certs/bjhqcdp03.dabsterinc.com.pem
cat bjhqcdp04.pem | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > certs/bjhqcdp04.dabsterinc.com.pem
cat bjhqcdp05.pem | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > certs/bjhqcdp05.dabsterinc.com.pem
```

**Worker Nodes:**
```bash
cat bjhqcdpw01.pem | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > certs/bjhqcdpw01.dabsterinc.com.pem
cat bjhqcdpw02.pem | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > certs/bjhqcdpw02.dabsterinc.com.pem
cat bjhqcdpw03.pem | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > certs/bjhqcdpw03.dabsterinc.com.pem
cat bjhqcdpw04.pem | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > certs/bjhqcdpw04.dabsterinc.com.pem
cat bjhqcdpw05.pem | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > certs/bjhqcdpw05.dabsterinc.com.pem
```

---

### Step 11 — Extract Private Keys

**Manager Nodes:**
```bash
cat bjhqcdpmgr01-key.pem | sed -ne '/-BEGIN PRIVATE KEY-/,/-END PRIVATE KEY-/p' > certs/bjhqcdpmgr01.dabsterinc.com-key.pem
cat bjhqcdpmgr02-key.pem | sed -ne '/-BEGIN PRIVATE KEY-/,/-END PRIVATE KEY-/p' > certs/bjhqcdpmgr02.dabsterinc.com-key.pem
```

**Data Nodes:**
```bash
cat bjhqcdp01-key.pem | sed -ne '/-BEGIN PRIVATE KEY-/,/-END PRIVATE KEY-/p' > certs/bjhqcdp01.dabsterinc.com-key.pem
cat bjhqcdp02-key.pem | sed -ne '/-BEGIN PRIVATE KEY-/,/-END PRIVATE KEY-/p' > certs/bjhqcdp02.dabsterinc.com-key.pem
cat bjhqcdp03-key.pem | sed -ne '/-BEGIN PRIVATE KEY-/,/-END PRIVATE KEY-/p' > certs/bjhqcdp03.dabsterinc.com-key.pem
cat bjhqcdp04-key.pem | sed -ne '/-BEGIN PRIVATE KEY-/,/-END PRIVATE KEY-/p' > certs/bjhqcdp04.dabsterinc.com-key.pem
cat bjhqcdp05-key.pem | sed -ne '/-BEGIN PRIVATE KEY-/,/-END PRIVATE KEY-/p' > certs/bjhqcdp05.dabsterinc.com-key.pem
```

**Worker Nodes:**
```bash
cat bjhqcdpw01-key.pem | sed -ne '/-BEGIN PRIVATE KEY-/,/-END PRIVATE KEY-/p' > certs/bjhqcdpw01.dabsterinc.com-key.pem
cat bjhqcdpw02-key.pem | sed -ne '/-BEGIN PRIVATE KEY-/,/-END PRIVATE KEY-/p' > certs/bjhqcdpw02.dabsterinc.com-key.pem
cat bjhqcdpw03-key.pem | sed -ne '/-BEGIN PRIVATE KEY-/,/-END PRIVATE KEY-/p' > certs/bjhqcdpw03.dabsterinc.com-key.pem
cat bjhqcdpw04-key.pem | sed -ne '/-BEGIN PRIVATE KEY-/,/-END PRIVATE KEY-/p' > certs/bjhqcdpw04.dabsterinc.com-key.pem
cat bjhqcdpw05-key.pem | sed -ne '/-BEGIN PRIVATE KEY-/,/-END PRIVATE KEY-/p' > certs/bjhqcdpw05.dabsterinc.com-key.pem
```

---

## Phase 4: Create Keystore

Run the following on **each host** individually (or loop over all hosts via SSH).

```bash
cd /opt/cloudera/security/pki
```

### Step 12 — Create PKCS12 Keystore

```bash
openssl pkcs12 -export \
  -inkey `hostname -f`-key.pem \
  -in `hostname -f`.pem \
  -certfile cacerts-chain.pem \
  -out `hostname -f`-keystore.pfx
```

### Step 13 — Convert PKCS12 to JKS Keystore

```bash
keytool -v -importkeystore \
  -srckeystore `hostname -f`-keystore.pfx \
  -srcstoretype PKCS12 \
  -destkeystore `hostname -f`-keystore.jks \
  -deststoretype JKS \
  -srcalias 1 \
  -destalias `hostname -f`
```

> **Note:** Steps 12 and 13 are currently documented for reference. Uncomment and execute when keystore creation is required for TLS configuration in Cloudera Manager.

---

## Host Reference

| Role | Hostname |
|------|----------|
| Manager Node 1 | bjhqcdpmgr01.dabsterinc.com |
| Manager Node 2 | bjhqcdpmgr02.dabsterinc.com |
| Data Node 1 | bjhqcdp01.dabsterinc.com |
| Data Node 2 | bjhqcdp02.dabsterinc.com |
| Data Node 3 | bjhqcdp03.dabsterinc.com |
| Data Node 4 | bjhqcdp04.dabsterinc.com |
| Data Node 5 | bjhqcdp05.dabsterinc.com |
| Worker Node 1 | bjhqcdpw01.dabsterinc.com |
| Worker Node 2 | bjhqcdpw02.dabsterinc.com |
| Worker Node 3 | bjhqcdpw03.dabsterinc.com |
| Worker Node 4 | bjhqcdpw04.dabsterinc.com |
| Worker Node 5 | bjhqcdpw05.dabsterinc.com |

---

## File Structure Reference

```
/var/tmp/SSL/CSR/
├── prodhosts.txt               # List of host FQDNs
├── SAN.cnf                     # OpenSSL SAN config template
├── tmp-sanconf/                # Per-host generated configs
│   └── <fqdn>-san.cnf
├── <fqdn>.csr                  # Certificate Signing Requests
├── <fqdn>.key                  # Private Keys
└── prod-csr.tar.gz             # Archive for CA submission

/opt/cloudera/security/pki/
├── rootca.pem                  # Root CA certificate
├── intca.pem                   # Intermediate CA certificate
├── cacerts-chain.pem           # Combined CA chain
├── <fqdn>.pem                  # Host certificate
├── <fqdn>-key.pem              # Host private key
├── <fqdn>-keystore.pfx         # PKCS12 keystore
└── <fqdn>-keystore.jks         # JKS keystore
```

---

*End of SOP*
