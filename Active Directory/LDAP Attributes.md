# LDAP Attributes



| LDAP Attributes  | openldap | AD | 
| ------------- | ------------- |
| LDAP Server Host  | openldap.dabsterinc.com  |  ad.dabsterinc.com |
| LDPP Server port  | 389 | 389, 636  |
|  User Object Class | posixAccount | user | 
| Username Attribute | uid | sAMAccountName |
| Group Object Class | posixGroup | group | 
| Group Name Attribute  | cn | cn |
| Group Member Attribute | memberUid | member |
| Distinguished name attribute | dn | distinguishedName |
| Search Base DN | dc=dabster,dc=com | DC=DASTERINC,DC=COM
| managerDn | cn=Manager,dc=hortonworks,dc=com | cn=adadmin,dc=hortonworks,dc=com |
|managerDnPassword | dabster123! | dabster123! |
