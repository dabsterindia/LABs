# Setup MIT Kerberos - (RHEL/CentOS/Oracle Linux Only)


## Perform below steps on KDC server

__Let's assume FQDN of my KDC server os `kdcmaster.dabsterinc.com`__

__Important:__
_here `dabsterinc.com` is the domain name, make a note of the domain name, this will be used in kerberos client configs_



## Step 1. Install new kerberos server packages on KDC:
```
yum install krb5-server krb5-libs krb5-workstation -y

```

## Step 2. Configure server specif files [kdc.conf](https://web.mit.edu/kerberos/krb5-1.12/doc/admin/conf_files/kdc_conf.html) & [kadm5.acl](https://web.mit.edu/kerberos/krb5-1.12/doc/admin/conf_files/kadm5_acl.html)

```
sed -i 's/EXAMPLE.COM/MIT.DABSTERINC.COM/g' /var/kerberos/krb5kdc/kdc.conf
echo '*/admin@MIT.DABSTERINC.COM	  *' > /var/kerberos/krb5kdc/kadm5.acl
```

```
cat /var/kerberos/krb5kdc/kdc.conf
cat /var/kerberos/krb5kdc/kadm5.acl
```

## Step 3. Configure krb5.conf

```
sed -i 's/#//g'  /etc/krb5.conf
sed -i 's/EXAMPLE.COM/MIT.DABSTERINC.COM/g'  /etc/krb5.conf
sed -i 's/example.com/dabsterinc.com/g'  /etc/krb5.conf

cat /etc/krb5.conf
```

Update `kdc =` & `admin_server =` with your KDC hostname

```
vi /etc/krb5.conf

```
Your fine krb5.conf should look similar to this:
```
[logging]
 default = FILE:/var/log/krb5libs.log
 kdc = FILE:/var/log/krb5kdc.log
 admin_server = FILE:/var/log/kadmind.log

[libdefaults]
 dns_lookup_realm = false
 ticket_lifetime = 24h
 renew_lifetime = 7d
 forwardable = true
 rdns = false
 pkinit_anchors = /etc/pki/tls/certs/ca-bundle.crt
 default_realm = MIT.DABSTERINC.COM
 default_ccache_name = KEYRING:persistent:%{uid}

[realms]
 MIT.DABSTERINC.COM = {
  kdc = kdcmaster.dabsterinc.com
  admin_server = kdcmaster.dabsterinc.com
 }

[domain_realm]
 .dabsterinc.com = MIT.DABSTERINC.COM
 dabsterinc.com = MIT.DABSTERINC.COM
```


## Step 4. Setup KDC database


## Step 5. Start Kerberos services






