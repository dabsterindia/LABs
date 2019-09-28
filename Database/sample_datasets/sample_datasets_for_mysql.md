

```
cd /tmp
wget http://www.mysqltutorial.org/wp-content/uploads/2018/03/mysqlsampledatabase.zip
unzip mysqlsampledatabase.zip
```

```
MYSQLUSER=root
MYSQLPASSWORD=root
MYSQL_HOST=localhost
```

```
mysql --user=$MYSQLUSER --password=$MYSQLPASSWORD --host=$MYSQL_HOST -e 'show databases'

mysql --user=$MYSQLUSER --password=$MYSQLPASSWORD --host=$MYSQL_HOST -e 'source /tmp/mysqlsampledatabase.sql;'

mysql --user=$MYSQLUSER --password=$MYSQLPASSWORD --host=$MYSQL_HOST  -e 'use classicmodels;show tables'

mysql --user=$MYSQLUSER --password=$MYSQLPASSWORD --host=$MYSQL_HOST  -e 'select * from classicmodels.customers limit 10'
```
