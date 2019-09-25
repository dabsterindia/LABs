

## Import sample data into Hive

#### 1: Login to hiveserver2 node and download sample data
```
cd /var/tmp/
mkdir sampledata

wget https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/labdata/sample_07.csv

wget https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/labdata/sample_08.csv
```

#### 2: Create sample database, create tables and load data in sample tables
```
# HIVE_CONNECTION_STRING="jdbc:hive2://zk1.hortonworks.com:2181,zk2.hortonworks.com:2181,zk3.hortonworks.com:2181/;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2"

# beeline -n hive -u $HIVE_CONNECTION_STRING
```

```
create database sampledb;
use sampledb;
```

```
CREATE TABLE `sample_07` (
`code` string ,
`description` string ,  
`total_emp` int ,  
`salary` int )
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' STORED AS TextFile;
```
```
load data local inpath '/var/tmp/sampledata/sample_07.csv' into table sample_07;
```
```
CREATE TABLE `sample_08` (
`code` string ,
`description` string ,  
`total_emp` int ,  
`salary` int )
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' STORED AS TextFile;
```
```
load data local inpath '/var/tmp/sampledata/sample_08.csv' into table sample_08;
```

#### 3: Query tables and see if data is loaded propperly
```
select * from sample_07 limit 10;

select * from sample_08 limit 10;

select count(*) from sample_07;
```
```
!q
```

