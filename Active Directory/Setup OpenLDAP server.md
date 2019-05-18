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


[root@dabsterinc ~]# netstat -antup | grep -i 389
```

### Step 3: Run below command to create an LDAP root password. (Here we have used "dabster123!")
```
[root@dabsterinc ~]# slappasswd -s dabster123!
{SSHA}7CF6juAoKseni3iy4FVrI+Lnh5dXuA4/
```
### Step 4: Create chrootpw.ldif file and specify the password generated above for "olcRootPW" section
```
[root@c2230-node1 ~]# vi chrootpw.ldif
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}7CF6juAoKseni3iy4FVrI+Lnh5dXuA4/
```

```
[root@dabsterinc ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f chrootpw.ldif 
```

### Step 5: Import basic Schemas
```
[root@dabsterinc ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif 
[root@dabsterinc ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif 
[root@dabsterinc ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif 
```

### Step 6:	Setup domain name on LDAP DB

##### 6.1 - Generate directory manager's password
```
[root@dabsterinc ~]# slappasswd -s dabster123!
{SSHA}zoOyuzjjy3TPLhen6LVWDCcRLER/Nsh4
```
##### 6.2 - Create chdomain ldif file
###### replace the password generated above for "olcRootPW" section
```
[root@dabsterinc ~]# vi chdomain.ldif
++++++++++++++++++++++++++++++++++++++++++++++++
dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth"
  read by dn.base="cn=Manager,dc=dabsterinc,dc=com" read by * none

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=dabsterinc,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=Manager,dc=dabsterinc,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}zoOyuzjjy3TPLhen6LVWDCcRLER/Nsh4

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcAccess
olcAccess: {0}to attrs=userPassword,shadowLastChange by
  dn="cn=Manager,dc=dabsterinc,dc=com" write by anonymous auth by self write by * none
olcAccess: {1}to dn.base="" by * read
olcAccess: {2}to * by dn="cn=Manager,dc=dabsterinc,dc=com" write by * read
```

##### 6.3 - Send the configuration to the LDAP server
```
[root@dabsterinc ~]# ldapmodify -Y EXTERNAL -H ldapi:/// -f chdomain.ldif 
```

##### 6.4 - Create base domain ldif file
```
[root@dabsterinc ~]# vi basedomain.ldif
```

```
dn: dc=dabsterinc,dc=com
objectClass: top
objectClass: dcObject
objectclass: organization
o: Dabsterinc
dc: Dabsterinc

dn: cn=Manager,dc=dabsterinc,dc=com
objectClass: organizationalRole
cn: Manager
description: Directory Manager

dn: ou=People,dc=dabsterinc,dc=com
objectClass: organizationalUnit
ou: People

dn: ou=Group,dc=dabsterinc,dc=com
objectClass: organizationalUnit
ou: Group

dn: ou=Users,dc=dabsterinc,dc=com
objectClass: organizationalUnit
ou: Users

dn: ou=Hadoop,dc=dabsterinc,dc=com
objectClass: organizationalUnit
ou: Hadoop
description: organizationalUnit for BigData & Hadoop
```
##### 6.5 - Send the configuration to the LDAP server
```
[root@dabsterinc ~]# ldapadd -x -D cn=Manager,dc=dabsterinc,dc=com -w 'dabster123!' -f basedomain.ldif
```


### Step 7: 
```
[root@dabsterinc ~]# ldapsearch -x -H ldap://localhost:389 -D 'cn=Manager,dc=dabsterinc,dc=com' -w 'dabster123!' -b 'dc=dabsterinc,dc=com'
```

------------------------------------------------------------------------------------------------------------------------------

## ADD Users and Groups

[Add users and groups trough CLI](https://github.com/dabsterindia/LABs/blob/master/Active%20Directory/openLdap%20-%20Commands.md "Most Useful commands in openLdap")


[Insytall and Setup phpLDAPAdmin](https://github.com/dabsterindia/LABs/blob/master/Active%20Directory/Install%20phpLDAPadmin.md "")
