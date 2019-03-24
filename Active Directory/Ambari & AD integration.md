# Integrate Ambari with AD to Sync AD users/groups

___Environment Details:___

Column 1 | Column 2
------------- | -------------
Operating System | CentOS Linux release 7.6.1810
HDP Stack Version | HDP-3.1
Ambari Version | Ambari-2.7.3
Directory Service | Windows AD Server 2012


___AD Details:___

Column 1 | Column 2
------------- | -------------
Hostname | hadoopad.australia-southeast1-b.c.silent-base-227511.internal
Base DN | DC=DABSTERINC,DC=COM
BindDN | CN=hdpadmin,OU=Users,OU=UAT,OU=BigData,OU=hadoop,DC=DABSTERINC,DC=COM
BindDN Password | Dabster@1


## Step 0: (Optional) Install openldap-client package to use ldapsearch
`# yum install openldap-clients -y`

```
# ldapsearch  -x -H ldaps://hadoopad.australia-southeast1-b.c.silent-base-227511.internal:3269 \
-D "CN=hdpadmin,OU=Users,OU=UAT,OU=BigData,OU=hadoop,DC=DABSTERINC,DC=COM" -w "Dabster@1" \
-b "OU=UAT,OU=BigData,OU=hadoop,DC=DABSTERINC,DC=COM"  -v > ldapsearch_result.out

# less ldapsearch_result.out
```

## Step 1. Setup LDAP for Ambari

`ambari-server setup-ldap`

```
Using python  /usr/bin/python
Currently 'no auth method' is configured, do you wish to use LDAP instead [y/n] (y)? y
Enter Ambari Admin login: admin
Enter Ambari Admin password: 

Primary LDAP Host:  hadoopad.australia-southeast1-b.c.silent-base-227511.internal
Primary LDAP Port:  389
Secondary LDAP Host <Optional>: 
Secondary LDAP Port <Optional>: 
Use SSL [true/false] (false):  false
User object class (user):  user
User ID attribute (sAMAccountName):  sAMAccountName
Group object class (group):  group
Group name attribute (cn):  cn
Group member attribute (member):  member
Distinguished name attribute (distinguishedName):  distinguishedName
Search Base (dc=ambari,dc=apache,dc=org):  OU=UAT,OU=BigData,OU=hadoop,DC=DABSTERINC,DC=COM
Referral method [follow/ignore] (follow):  ignore
Bind anonymously [true/false] (false):  false
Handling behavior for username collisions [convert/skip] for LDAP sync (skip):  skip
Force lower-case user names [true/false]: false
Results from LDAP are paginated when requested [true/false]: true
ambari.ldap.connectivity.bind_dn: CN=hdpadmin,OU=Users,OU=UAT,OU=BigData,OU=hadoop,DC=DABSTERINC,DC=COM
ambari.ldap.connectivity.bind_password: *****
Save settings [y/n] (y)? y
Saving LDAP properties...
Saving LDAP properties finished
Ambari Server 'setup-ldap' completed successfully.
```

## Step 2. Create two files and add AD usersnames & groupnames
```
# cat /var/tmp/users.txt
user1,user2,user3

# cat /var/tmp/groups.txt
group1,group2,group3
```

## Step 3. Sync specific users & groups from AD

```
ambari-server sync-ldap

```

```
Using python  /usr/bin/python
Syncing with LDAP...
Enter Ambari Admin login: admin
Enter Ambari Admin password: 

Fetching LDAP configuration from DB.
Syncing all..........

Completed LDAP Sync.
Summary:
  memberships:
    removed = 0
    created = 2
  users:
    skipped = 0
    removed = 0
    updated = 0
    created = 3
  groups:
    updated = 0
    removed = 0
    created = 3

Ambari Server 'sync-ldap' completed successfully.
```

## Step 4. Login ambari UI and verify whether all users & groups are syncd
Ambari UI --> admin --> Manage Ambari --> Users
