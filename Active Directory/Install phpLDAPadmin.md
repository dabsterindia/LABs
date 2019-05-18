# Install & Setup phpLDAPadmin on CentOS 7


### 1. Update system packages
```
# yum update && sudo yum upgrade -y
```
### 2. Install extra PHP packages
```
# yum install php-ldap php-mbstring php-pear php-xml -y
```
### 3. Install Extra Packages for Enterprise Linux (EPEL) release
```
# yum install epel-release
```

### 4. Start LDAP services
```
# systemctl start slapd && sudo systemctl enable slapd
```

