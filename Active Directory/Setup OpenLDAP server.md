# Configure LDAP Server (OpenLDAP)



### Step 1: Install LDAP packages on LDAP server
```
[root@dabsterinc ~]# yum -y install openldap-servers openldap-clients
[root@dabsterinc ~]# cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG 
[root@dabsterinc ~]# chown ldap. /var/lib/ldap/DB_CONFIG 
```

### Step 2: Start & verify LDAP server
```
[root@dabsterinc ~]# systemctl start slapd 
[root@dabsterinc ~]# systemctl enable slapd


netstat -antup | grep -i 389
```

### Step 3: Run below command to create an LDAP root password. (Here we have used "dabster123!")
```
[root@dabsterinc ~]# slappasswd -s dabster123!
{SSHA}7CF6juAoKseni3iy4FVrI+Lnh5dXuA4/
```
### Step 4: Create chrootpw.ldif file and specify the password generated above for "olcRootPW" section
```
[root@c2230-node1 ~]# cd /var/tmp; vi chrootpw.ldif
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}7CF6juAoKseni3iy4FVrI+Lnh5dXuA4/
```

```
ldapadd -Y EXTERNAL -H ldapi:/// -f chrootpw.ldif 
```

### step 5: Import basic Schemas.
```
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif 
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif 
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif 
```
