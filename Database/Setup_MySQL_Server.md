# Install and Setup Mysql/MariaDB serverb database on CentOS/RHEL


### 1. Install mysql-server package
```
$ yum install mysql-server -y
```

### 2. Start mysql deamon
```
$ service mysqld start
$ chkconfig mysqld on
```

### 3. Login & Test Mysql
```
$ mysql

mysql> show databases;
mysql> create database testdb;
mysql> use testdb;
mysql> CREATE TABLE Persons(id int, name varchar(255));
mysql> show tables;
```

### 4. Create a user and grant privilleges
```
$ mysql
CREATE USER 'mysqldba'@'%' IDENTIFIED BY 'mysqldba';
GRANT ALL PRIVILEGES ON *.* TO 'mysqldba'@'%';
CREATE USER 'mysqldba'@'localhost' IDENTIFIED BY 'mysqldba';
GRANT ALL PRIVILEGES ON *.* TO 'mysqldba'@'localhost';
CREATE USER 'mysqldba'@'mysql.dabsterinc.com' IDENTIFIED BY 'mysqldba';
GRANT ALL PRIVILEGES ON *.* TO 'mysqldba'@'mysql.dabsterinc.com';
FLUSH PRIVILEGES;
exit;
```

```
$ mysql -u mysqldba -p
$ show databases;
```

### 5. (Optional) See active connections
```
$ lsof -i :3306
```
