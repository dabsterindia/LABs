# Ranger Installation using Ambari


## Pre-requisites:
  * Ranger uses a Database to store its policies & user information. 
  * Install a new database OR Use an existing one.


#### Setup MySQL Server

### a. Install MySQL Server (Optional - Skip this step if you already have a database installed)

```
yum install -y mysql-server
service mysqld start
chkconfig mysqld on
```

### b. Login Mysql and create an admin user for ranger. Here we are creating a user `rangerdba` with password `rangerdba`

```
# mysql
```

```
CREATE USER 'rangerdba'@'localhost' IDENTIFIED BY 'rangerdba';
CREATE USER 'rangerdba'@'%' IDENTIFIED BY 'rangerdba';
GRANT ALL PRIVILEGES ON *.* TO 'rangerdba'@'localhost';
GRANT ALL PRIVILEGES ON *.* TO 'rangerdba'@'%';
GRANT ALL PRIVILEGES ON *.* TO 'rangerdba'@'localhost' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'rangerdba'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
exit
```
```
# mysql -urangerdba -prangerdba
```

### c. Setup ambari-server to use mysql-connector while connenting to MySQL server

On Ambari-Server hosts, Verify whether mysql connector is installed & available in below location.
```
ls /usr/share/java/mysql-connector-java.jar
```

If connector is missing from above location, Install mysql connector using below command.
```
yum install -y mysql-connector-java*
```
Set JDBC driver path. Run below command on ambari server
```
ambari-server setup --jdbc-db=mysql --jdbc-driver=/usr/share/java/mysql-connector-java.jar
```


### Install Ranger



