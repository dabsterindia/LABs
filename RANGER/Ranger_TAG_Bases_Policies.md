


### 1) Download script & generate sample data file
```
yum clean all
yum install git -y

cd /var/tmp/
git clone https://github.com/ansarigulshad/script_datagenerator.git
cd script_datagenerator
chmod +x *.sh
```
```
sh data_generator.sh
```
```
head -50 employee_data.csv
```
### 2) Create Hive database & table

#### a. Login to hiveserver2 node and login beeline (change hive connection string as per your setup)
```
# HIVE_CONNECTION_STRING="jdbc:hive2://zk1.hortonworks.com:2181,zk2.hortonworks.com:2181,zk3.hortonworks.com:2181/;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2"

# beeline -n hive -u $HIVE_CONNECTION_STRING
```
#### b. Create database
```
create database emp_sampledb;
use emp_sampledb;
```
#### c. Create Table
```
create table employee(id int, name string, salary int, city string,mobile_no bigint) 
row format delimited
fields terminated by ','
stored as textfile;
```
```
!q
```
#### d. Load data to `employee` table
```
sudo -su hive hdfs dfs -put \
/var/tmp/script_datagenerator/employee_data.csv \
/warehouse/tablespace/managed/hive/emp_sampledb.db/employee
```
#### e. Check employee table
```
# beeline -n hive -p hive -e "select * from emp_sampledb.employee limit 50" -u $HIVE_CONNECTION_STRING
```

### 3. Install Atlas through ambari


#### 0. Prerequisites : Before you begin, make sure you have following services installed in your HDP cluster
* Install infra-solr
* Install HBase
* Install Kafka

#### i. Add Atlas service

#### ii. Set password for admin user at "Customize services"

#### iii. Complete the Atlas installation

#### iv. Restart All required













