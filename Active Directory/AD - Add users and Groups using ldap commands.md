# Add users/groups in Active Directory using ldap commands

### Set Variables
```
ARG_DOMAIN=DABSTERINC.COM
ARG_DOMAINCONTROLLER=DC=DABSTERINC,DC=COM
ARG_LDAPURI=ldaps://adserver.asia-southeast1-b.c.x-plateau-236613.internal:636
ARG_BINDDN=CN=Administrator,CN=Users,DC=DABSTERINC,DC=COM
ARG_USERPSWD=Hadoop123!
ARG_SEARCHBASE=OU=Hadoop,DC=DABSTERINC,DC=COM

ARG_NewUserPass=`echo -n '"Hadoop123!"' | iconv -f UTF8 -t UTF16LE | base64 -w 0`
```

### 1) Add User
#### a) Create ldif file for user with user attributes
The first part of the following LDIF creates the disabled user account, the second part sets the password and the last part enables the account:

```
# cat > /tmp/Prutser.ldif <<EOFILE
dn: CN=Piet Prutser,${ARG_SEARCHBASE}
changetype: add
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: Gulshad Ansari
sn: Ansari
givenName: Gulshad
displayName: Gulshad Ansari
name: Gulshad Ansari
accountExpires: 9223372036854775807
userAccountControl: 514
sAMAccountName: gansari
userPrincipalName: gansari@${ARG_DOMAIN}

dn: CN=Gulshad Ansari,${ARG_SEARCHBASE}
changetype: modify
replace: unicodePwd
unicodePwd::${ARG_NewUserPass}

dn: CN=Gulshad Ansari,${ARG_SEARCHBASE}
changetype: modify
replace: userAccountControl
userAccountControl: 512
EOFILE
```
#### b) Add user entry
```
# LDAPTLS_REQCERT=never ldapadd -x -H "${ARG_LDAPURI}" -a -D "${ARG_BINDDN}" -w "${ARG_USERPSWD}" -f /tmp/adduser-gansari.ldif 
```
#### c) Search and verify entries
```
# LDAPTLS_REQCERT=never ldapsearch -x -H "${ARG_LDAPURI}" -D "${ARG_BINDDN}" -b "${ARG_SEARCHBASE}"  -L -w "${ARG_USERPSWD}" "cn=Piet Prutser"
```

### 2) Add Group
#### a) Create ldif file with group attributes
```
# cat > /tmp/mygroup.ldif <<EOFILE
dn: CN=mygroup,OU=Groups,OU=Hadoop,DC=DABSTERINC,DC=COM
objectClass: top
objectClass: group
cn: mygroup
name: mygroup
distinguishedName: CN=mygroup,OU=Groups,OU=Hadoop,DC=DABSTERINC,DC=COM
instanceType: 4
sAMAccountName: mygroup
gidNumber: 666
EOFILE
```
#### b) Add group entry
```
# LDAPTLS_REQCERT=never ldapadd -x -H "${ARG_LDAPURI}" -a -D "${ARG_BINDDN}" -w "${ARG_USERPSWD}" -f /tmp/mygroup.ldif 
```
```
# LDAPTLS_REQCERT=never ldapsearch -x -H "${ARG_LDAPURI}" -D "${ARG_BINDDN}" -b "${ARG_SEARCHBASE}"  -L -w "${ARG_USERPSWD}" "cn=mygroup"
```

### 3) Add users in group
#### a) Create ldif file to add users in  group
```
# cat > /tmp/adduserstomygroup.ldif <<EOFILE
dn: CN=mygroup,OU=Groups,OU=Hadoop,DC=DABSTERINC,DC=COM
changetype: modify
add: member
member: CN=Piet Prutser,OU=Users,OU=Hadoop,DC=DABSTERINC,DC=COM
member: CN=Wasim Khan,OU=Users,OU=Hadoop,DC=DABSTERINC,DC=COM
member: CN=s-admin,OU=Users,OU=Hadoop,DC=DABSTERINC,DC=COM
member: CN=Jason,OU=Users,OU=Hadoop,DC=DABSTERINC,DC=COM
EOFILE
```

#### b) Modify entries
```
# LDAPTLS_REQCERT=never ldapadd -x -H "${ARG_LDAPURI}" -a -D "${ARG_BINDDN}" -w "${ARG_USERPSWD}" -f /tmp/adduserstomygroup.ldif 
```
```
# LDAPTLS_REQCERT=never ldapsearch -x -H "${ARG_LDAPURI}" -D "${ARG_BINDDN}" -b "${ARG_SEARCHBASE}"  -L -w "${ARG_USERPSWD}" "cn=mygroup"
```

__You can also add users in the group wile creating a new group.__
```
cat > /tmp/mynewgroup.ldif <<EOFILE
dn: CN=mynewgroup,OU=Groups,OU=Hadoop,DC=DABSTERINC,DC=COM
objectClass: top
objectClass: group
cn: mynewgroup
name: mynewgroup
distinguishedName: CN=mynewgroup,OU=Groups,OU=Hadoop,DC=DABSTERINC,DC=COM
instanceType: 4
sAMAccountName: mygroup
gidNumber: 666

dn: CN=mynewgroup,OU=Groups,OU=Hadoop,DC=DABSTERINC,DC=COM
changetype: modify
add: member
member: CN=My Testuser,OU=Users,OU=Hadoop,DC=DABSTERINC,DC=COM
member: CN=s-admin,OU=Users,OU=Hadoop,DC=DABSTERINC,DC=COM
member: CN=Jason,OU=Users,OU=Hadoop,DC=DABSTERINC,DC=COM
EOFILE
```
```
# LDAPTLS_REQCERT=never ldapadd -x -H "${ARG_LDAPURI}" -a -D "${ARG_BINDDN}" -w "${ARG_USERPSWD}" -f /tmp/mynewgroup.ldif 
```



