
## Create sample hive table(s) using sample population dataset

#### 1: Login to hiveserver2 node and download sample data

```
cd /var/tmp/
wget https://raw.githubusercontent.com/ansarigulshad/Datasets/master/cities.csv
```

#### 2: Create sample database (population), create table (cities) and load data in `cities` table

```
# HIVE_CONNECTION_STRING="jdbc:hive2://zk1.hortonworks.com:2181,zk2.hortonworks.com:2181,zk3.hortonworks.com:2181/;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2"

# beeline -n hive -u $HIVE_CONNECTION_STRING
```
```
create database population;
use population;
```
```
CREATE TABLE `cities` (
`rank` int,
`name` string,
`population_2011` string,
`population_2001` string,
`state` string )
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' STORED AS TextFile;
```
```
load data local inpath '/var/tmp/cities.csv' into table cities;
```
```
select * from cities limit 50;
```
```
desc cities;
```
```
describe formatted cities;
```
#### 3. Create two new tables `cities_pop_2001` & `cities_pop_2011` using existing table
```
create table cities_pop_2001 as select rank,name,population_2001,state from cities;
```
```
create table cities_pop_2011 as select rank,name,population_2011,state from cities;
```
```
show tables;
```

