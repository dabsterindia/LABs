# Reset the root password for MySQL

### Step 1. Stop Mysql Service

```
# service mysqld stop
# service mysqld status
```
### Step 2. Start Mysql server in safe mode
```
# sudo mysqld_safe –skip-grant-tables &
# service mysqld status
```
### Step 3. Login as root
```
# mysql -u root
```
### Step 4. Setup new password for root
```
mysql>  use mysql;
mysql>  update user set Password=PASSWORD(“newpass”) where User=’root’;
mysql>  flush privileges;
mysql>  quit;
```
### Step 5. Restart Mysql service 
```
# service mysqld stop
# service mysqld start
```
### Step 6. Now login with new password
```
# mysql -u root -p
```
