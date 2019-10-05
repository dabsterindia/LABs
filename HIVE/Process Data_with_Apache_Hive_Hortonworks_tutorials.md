# Process Data with Apache Hive


#### Login to hiveserver2 node and download sample data

```
cd /var/tmp/
wget https://raw.githubusercontent.com/hortonworks/data-tutorials/master/tutorials/hdp/how-to-process-data-with-apache-hive/assets/driver_data.zip
```
```
unzip driver_data.zip
```

#### Create sample database (hwx_sampledb), create tables (drivers & timesheet) and load data

```
# HIVE_CONNECTION_STRING="jdbc:hive2://zk1.hortonworks.com:2181,zk2.hortonworks.com:2181,zk3.hortonworks.com:2181/;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2"

# beeline -n hive -u $HIVE_CONNECTION_STRING
```
```
create database hwx_sampledb;
use hwx_sampledb;
```

```
CREATE TABLE temp_drivers (col_value STRING) STORED AS TEXTFILE;
```
```
CREATE TABLE temp_timesheet (col_value string) STORED AS TEXTFILE;
```

```
LOAD DATA LOCAL INPATH '/var/tmp/driver_data/drivers.csv' INTO TABLE temp_drivers;

LOAD DATA INPATH '/var/tmp/driver_data/timesheet.csv'  INTO TABLE temp_timesheet;
```

```
SHOW tables;

SELECT * FROM temp_drivers;

SELECT * FROM temp_timesheet LIMIT 10;
```

```
CREATE TABLE drivers (driverId INT, name STRING, ssn BIGINT,
                      location STRING, certified STRING, wageplan STRING);
```

```
insert overwrite table drivers
SELECT
  regexp_extract(col_value, '^(?:([^,]*),?){1}', 1) driverId,
  regexp_extract(col_value, '^(?:([^,]*),?){2}', 1) name,
  regexp_extract(col_value, '^(?:([^,]*),?){3}', 1) ssn,
  regexp_extract(col_value, '^(?:([^,]*),?){4}', 1) location,
  regexp_extract(col_value, '^(?:([^,]*),?){5}', 1) certified,
  regexp_extract(col_value, '^(?:([^,]*),?){6}', 1) wageplan
from temp_drivers;
```

```
SELECT * FROM drivers;
```

```
CREATE TABLE timesheet (driverId INT, week INT, hours_logged INT , miles_logged INT);
```

```
insert overwrite table timesheet
SELECT
  regexp_extract(col_value, '^(?:([^,]*),?){1}', 1) driverId,
  regexp_extract(col_value, '^(?:([^,]*),?){2}', 1) week,
  regexp_extract(col_value, '^(?:([^,]*),?){3}', 1) hours_logged,
  regexp_extract(col_value, '^(?:([^,]*),?){4}', 1) miles_logged
from temp_timesheet;
```

```
SELECT * FROM timesheet LIMIT 10;
```


#### Create Query to Filter The Data (driverId, hours_logged, miles_logged)
```
SELECT driverId, sum(hours_logged), sum(miles_logged) FROM timesheet GROUP BY driverId;
```

#### Create Query to Join The Data (driverId, name, hours_logged, miles_logged);
```
SELECT d.driverId, d.name, t.total_hours, t.total_miles from drivers d
JOIN (SELECT driverId, sum(hours_logged)total_hours, sum(miles_logged)total_miles FROM timesheet GROUP BY driverId ) t
ON (d.driverId = t.driverId);


