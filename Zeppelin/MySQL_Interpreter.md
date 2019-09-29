## CONNECTING APACHE ZEPPELIN TO MYSQL

### 1. Download MySQL Connector
Download the MySQL Connector JAR from https://dev.mysql.com/downloads/connector/j/
```
cd /tmp
wget  https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.41.tar.gz
```
### 2. Move MySQL Connector into Interpreter Directory

```
tar xzvf mysql-connector-java-5.1.41.tar.gz
cp mysql-connector-java-5.1.41/mysql-connector-java-5.1.41-bin.jar /usr/hdp/current/zeppelin-server/interpreter/jdbc/
```

### 3. Create a MySQL Interpreter

a. Navigate to the Zeppelin interpreter page. (Zeppelin UI > admin > Interpreter) OR (http://zeppelin-host:9995/#/interpreter)

b. Click the “+Create” button to create a new interpreter.

c. Give your interpreter a name `mysql`

d. Select the `jdbc` interpreter group

e. Configure your MySQL JDBC connection

```
default.driver = com.mysql.jdbc.Driver
default.user   = (username used to login)
default.pw     = (password used to login)
default.url    =  jdbc:mysql://mysqlhost:3306/
```

f. In the `Dependencies` section, you must specify the artifact of the MySQL Connector JAR that we previously downloaded. Since we downloaded version 5.1.41 in this example, the artifact is `mysql:mysql-connector-java:5.1.41`

_Example:_
![alt text](https://github.com/dabsterindia/LABs/blob/master/tmp/images/zeppelin_mysql_interpretter_settings.png)


### 4. Create a new notebook, and use `mysql` as your interpreter.
![alt text](https://github.com/dabsterindia/LABs/blob/master/tmp/images/zeppelin_create_notebook_mysql.png)

### 5. Create a sample dataset on MYSQL (optional)
Follow https://github.com/dabsterindia/LABs/blob/master/Database/sample_datasets/sample_datasets_for_mysql.md

### 6. Execute MYSQL Queries from Zeppelin Notebook
![alt text](https://github.com/dabsterindia/LABs/blob/master/tmp/images/zeppelin_mysql_%20interpreter_sample_sql1.png)

![alt text](https://github.com/dabsterindia/LABs/blob/master/tmp/images/zeppelin_mysql_%20interpreter_sample_sql2.png)
