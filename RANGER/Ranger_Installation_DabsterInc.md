# Ranger Installation using Ambari


## Pre-requisites:
  * Install a database (MySQL, PG, ORACLE etc) OR Use an existing one
  * Install Infra Solr (for audits)


### Setup MySQL Server

#### a. Install MySQL Server (Optional - Skip this step if you already have a database installed)

```
yum install -y mysql-server
service mysqld start
chkconfig mysqld on
```

#### b. Login Mysql and create an admin user for ranger. Here we are creating a user `rangerdba` with password `rangerdba`

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

#### c. Setup ambari-server to use mysql-connector while connenting to MySQL server

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


## Install Ranger
#### a. Go to Ambari Dashboard --> Add Service --> Select Ranger

#### b. Choose Service > Ranger > Next

#### c. Check the prerequisites as below and check the box after all the requirements are met --> Click "Proceed"

#### d. Assign Masters > Select host for Ranger admin & Ranger User Sync > Next

#### e. Assign Slaves and Clients > Select host for Ranger tag Sync > Next

#### f. Customize Services update the necessary information


#### g. Review & deploy



