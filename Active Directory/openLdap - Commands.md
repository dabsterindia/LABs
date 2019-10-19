# The Most Common openLDAP Commands

## ADD Users:

```
# vi user.ldif
```
```
dn: uid=user1,ou=Hadoop,dc=hortonworks,dc=com
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

```
# ldapadd -x -H ldap://localhost:389 -D cn=Manager,dc=hortonworks,dc=com -W -f user.ldif
```

### EXTRA's - Create more users using below steps
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
ldapadd -x -D "cn=Manager,dc=hortonworks,dc=com" -f user2.ldif -w hadoop123!
ldapadd -x -D "cn=Manager,dc=hortonworks,dc=com" -f user3.ldif -w hadoop123!
ldapadd -x -D "cn=Manager,dc=hortonworks,dc=com" -f jason.ldif -w hadoop123!
ldapadd -x -D "cn=Manager,dc=hortonworks,dc=com" -f john.ldif -w hadoop123!
ldapadd -x -D "cn=Manager,dc=hortonworks,dc=com" -f jassi.ldif -w hadoop123!
ldapadd -x -D "cn=Manager,dc=hortonworks,dc=com" -f gansari.ldif -w hadoop123!

# Set Password

ldappasswd -s Welcome@1 -D "cn=Manager,dc=hortonworks,dc=com" -x "uid=user1,ou=Hadoop,dc=hortonworks,dc=com" -w hadoop123!
ldappasswd -s Welcome@1 -D "cn=Manager,dc=hortonworks,dc=com" -x "uid=user2,ou=Hadoop,dc=hortonworks,dc=com" -w hadoop123!
ldappasswd -s Welcome@1 -D "cn=Manager,dc=hortonworks,dc=com" -x "uid=user3,ou=Hadoop,dc=hortonworks,dc=com" -w hadoop123!
ldappasswd -s Welcome@1 -D "cn=Manager,dc=hortonworks,dc=com" -x "uid=jason,ou=Hadoop,dc=hortonworks,dc=com" -w hadoop123!
ldappasswd -s Welcome@1 -D "cn=Manager,dc=hortonworks,dc=com" -x "uid=john,ou=Hadoop,dc=hortonworks,dc=com" -w hadoop123!
ldappasswd -s Welcome@1 -D "cn=Manager,dc=hortonworks,dc=com" -x "uid=jassi,ou=Hadoop,dc=hortonworks,dc=com" -w hadoop123!
ldappasswd -s Welcome@1 -D "cn=Manager,dc=hortonworks,dc=com" -x "uid=gansari,ou=Hadoop,dc=hortonworks,dc=com" -w hadoop123!
```


## ADD GROUP

```
# vi groupCreate.ldif
```

```
dn: cn=hd-admins,ou=Hadoop,dc=hortonworks,dc=com
objectClass: top
objectClass: posixGroup
gidNumber: 678
```

```
# ldapadd -x -D "cn=Manager,dc=hortonworks,dc=com" -w 'hadoop123!' -f groupCreate.ldif
```


## ADD Users to a Group:
```
# vi addusertogroup.ldif
```

```
dn: cn=hd-admins,ou=Hadoop,dc=hortonworks,dc=com
changetype: modify
add: memberuid
memberuid: user1

dn: cn=hd-admins,ou=Hadoop,dc=hortonworks,dc=com
changetype: modify
add: memberuid
memberuid: user2

dn: cn=hd-admins,ou=Hadoop,dc=hortonworks,dc=com
changetype: modify
add: memberuid
memberuid: user1
memberuid: john
```

```
# ldapmodify -x -D "cn=Manager,dc=hortonworks,dc=com" -w 'hadoop123!' -f addusertogroup.ldif
```

## ADD Organizational Unit (OU)
```
# vi newOU.ldif
```

```
dn: ou=Dev,ou=Hadoop,dc=hortonworks,dc=com
objectClass: organizationalUnit
ou: kerberos
description: kerberos principals OU
```

```
# ldapadd -x -D cn=Manager,dc=hortonworks,dc=com -w 'hadoop123!' -f newOU.ldif
```

## Delete Entry
```
ldapdelete -W -D "cn=Manager,dc=hortonworks,dc=com" "uid=john,ou=People,dc=hortonworks,dc=com"
```

### ldap Search
```
# ldapsearch -x cn=user1 -b dc=hortonworks,dc=com
```
```
# ldapsearch -x -H ldap://$(hostname -f):389 -D cn=Manager,dc=hortonworks,dc=com -w 'hadoop123!' -b dc=hortonworks,dc=com
```
```
# ldapsearch -x -H ldap://$(hostname -f):389 -D cn=Manager,dc=hortonworks,dc=com -w 'hadoop123!' -b dc=hortonworks,dc=com cn=john
```
