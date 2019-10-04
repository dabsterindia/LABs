
## Create sample hive table (fifa_19_players) using sample payers dataset

#### 1: Login to hiveserver2 node and download sample data

```
cd /var/tmp/
wget https://raw.githubusercontent.com/ansarigulshad/Datasets/master/fifa_19_players_data.csv
```

#### 2: Create sample database (fifadb), create table (fifa_19_players) and load data in `fifa_19_players` table

```
# HIVE_CONNECTION_STRING="jdbc:hive2://zk1.hortonworks.com:2181,zk2.hortonworks.com:2181,zk3.hortonworks.com:2181/;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2"

# beeline -n hive -u $HIVE_CONNECTION_STRING
```
```
create database fifadb;
use fifadb;
```
```
CREATE TABLE `fifa_19_players` (
`sr_no` int ,
`id` int ,  
`name` string ,  
`age` int,
`nationality` string,
`club` string,
`preferred_foot` string,
`position` string,
`jersey_number` int,
`joined` int)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' STORED AS TextFile;
```
```
show tables;
```
```
load data local inpath '/var/tmp/fifa_19_players_data.csv' into table fifa_19_players;
```
```
select * from fifa_19_players limit 50;
```
```
desc fifa_19_players;
```
```
describe formatted fifa_19_players;
```
```
select * from fifa_19_players_dataset limit 20;
```
```
select * from fifa_19_players_dataset where club='Real Madrid';
```

```
SELECT DISTINCT nationality from fifa_19_players;
```

```
select count(*) from fifa_19_players_dataset;
```

```
select count(*) from fifa_19_players where nationality='Russia';
```
