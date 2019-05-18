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
ldapmodify -Y EXTERNAL -H ldapi:/// -f chdomain.ldif 
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
ADD Users:

### Create User ldif file to add new user
```
[root@dabsterinc ~]# vi user.ldif
dn: uid=user1,ou=Hadoop,dc=dabsterinc,dc=com
objectClass: top
objectClass: account
objectClass: posixAccount
objectClass: shadowAccount
cn: user1
uid: user1
uidNumber: 9999
gidNumber: 100
homeDirectory: /home/user1
loginShell: /bin/bash
gecos: user1
userPassword: {crypt}x
shadowLastChange: 17058
shadowMin: 0
shadowMax: 99999
shadowWarning: 7
```

### Add user1
```
[root@dabsterinc ~]# ldapadd -x -H ldap://localhost:389 -D cn=Manager,dc=dabsterinc,dc=com -W -f user.ldif
```

### Create more users if required using below steps
```
cp user.ldif user2.ldif
cp user.ldif user3.ldif
cp user.ldif jason.ldif 
cp user.ldif john.ldif 
cp user.ldif jassi.ldif
cp user.ldif gansari.ldif

sed -i s/user1/user2/g user2.ldif
sed -i s/user1/user3/g user3.ldif
sed -i s/user1/jason/g jason.ldif
sed -i s/user1/john/g john.ldif
sed -i s/user1/jassi/g jassi.ldif
sed -i s/user1/gansari/g gansari.ldif

# Add User
ldapadd -x -D "cn=Manager,dc=dabsterinc,dc=com" -f user2.ldif -w dabster123!
ldapadd -x -D "cn=Manager,dc=dabsterinc,dc=com" -f user3.ldif -w dabster123!
ldapadd -x -D "cn=Manager,dc=dabsterinc,dc=com" -f jason.ldif -w dabster123!
ldapadd -x -D "cn=Manager,dc=dabsterinc,dc=com" -f john.ldif -w dabster123!
ldapadd -x -D "cn=Manager,dc=dabsterinc,dc=com" -f jassi.ldif -w dabster123!
ldapadd -x -D "cn=Manager,dc=dabsterinc,dc=com" -f gansari.ldif -w dabster123!

# Set Password

ldappasswd -s Welcome@1 -D "cn=Manager,dc=dabsterinc,dc=com" -x "uid=user1,ou=Hadoop,dc=dabsterinc,dc=com" -w dabster123!
ldappasswd -s Welcome@1 -D "cn=Manager,dc=dabsterinc,dc=com" -x "uid=user2,ou=Hadoop,dc=dabsterinc,dc=com" -w dabster123!
ldappasswd -s Welcome@1 -D "cn=Manager,dc=dabsterinc,dc=com" -x "uid=user3,ou=Hadoop,dc=dabsterinc,dc=com" -w dabster123!
ldappasswd -s Welcome@1 -D "cn=Manager,dc=dabsterinc,dc=com" -x "uid=jason,ou=Hadoop,dc=dabsterinc,dc=com" -w dabster123!
ldappasswd -s Welcome@1 -D "cn=Manager,dc=dabsterinc,dc=com" -x "uid=john,ou=Hadoop,dc=dabsterinc,dc=com" -w dabster123!
ldappasswd -s Welcome@1 -D "cn=Manager,dc=dabsterinc,dc=com" -x "uid=jassi,ou=Hadoop,dc=dabsterinc,dc=com" -w dabster123!
ldappasswd -s Welcome@1 -D "cn=Manager,dc=dabsterinc,dc=com" -x "uid=gansari,ou=Hadoop,dc=dabsterinc,dc=com" -w dabster123!
```

### ldap Search
```
ldapsearch -x cn=user1 -b dc=dabsterinc,dc=com
```
```
ldapsearch -x -H ldap://$(hostname -f):389 -D cn=Manager,dc=dabsterinc,dc=com -w 'dabster123!' -b dc=dabsterinc,dc=com
```
```
ldapsearch -x -H ldap://$(hostname -f):389 -D cn=Manager,dc=dabsterinc,dc=com -w 'dabster123!' -b dc=dabsterinc,dc=com cn=john
```

## ADD GROUP

#### Create LDIF file for New Group
```
# vi group1.ldif
```

```
dn: cn=hd-admins,ou=Hadoop,dc=dabsterinc,dc=com
objectClass: top
objectClass: posixGroup
gidNumber: 678
```

```
# ldapadd -x -D "cn=Manager,dc=dabsterinc,dc=com" -w 'dabster123!' -f group1.ldif
```


## Add an existing user to a group:
```
# vi addusertogroup.ldif
```

```
dn: cn=hd-admins,ou=Hadoop,dc=dabsterinc,dc=com
changetype: modify
add: memberuid
memberuid: user1
memberuid: john
```

```
# ldapmodify -x -D "cn=Manager,dc=dabsterinc,dc=com" -w 'dabster123!' -f addusertogroup.ldif
```

### 2. 
`vi group2.ldif`
```
dn: cn=hd-dev,ou=Hadoop,dc=dabsterinc,dc=com
objectClass: top
objectClass: posixGroup
gidNumber: 678
```
```
# ldapadd -x -D "cn=Manager,dc=dabsterinc,dc=com" -w 'dabster123!' -f group2.ldif
```

