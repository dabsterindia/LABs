# Multi Node Apache Hadoop - Cluster Deployement on Google Cloud Platform (GCP)

___Environment Details:___

1 | 2
------------- | -------------
Operating System | Ubuntu 14.04
Java Vesion | OpenJDK 8
Hadoop Version | Hadoop-2.7.4

## Step 1. Preparing the Environment (Prerequisites)
#### FOR UBUNTU:
#### i. Configure `/etc/hosts` file
#### ii. Configure passwordless ssh
#### iii. Verify firewall status, it must be disabled

#### For RHEL/Centos:
Follow [Hadoop Prerequisites](https://github.com/dabsterindia/LABs/blob/master/Cluster%20Deployments/Hadoop_Prerequisites.md "Hadoop Prerequisites") article to setup the environment for hadoop deployement


## Step 2. Install dsh and Update machine.list file
#### 2.1. Install dsh

```
sudo apt-get update
sudo apt-get install dsh -y
```

#### 2.2. Update machine.list file with all host entries. Comment out or remove localhost.

```
sudo vi /etc/dsh/machines.list
```

```
#localhost
nn.us-central1-a.c.lithe-grid-257118.internal
snn.us-central1-a.c.lithe-grid-257118.internal
dn1.us-central1-a.c.lithe-grid-257118.internal
dn2.us-central1-a.c.lithe-grid-257118.internal
dn3.us-central1-a.c.lithe-grid-257118.internal
```

#### 2.3. Verify dsh

`dsh -a "hostname;echo "------------"`

## Step 3. Install JAVA 8 (openJDK) on all nodes

```
dsh -a 'sudo apt update'
dsh -a 'sudo apt install openjdk-8-jdk openjdk-8-jre openjdk-8-jdk-headless -y'
```

## Step 4. Install Hadoop:

#### Install Hadoop on all nodes using dsh utility, untar it and move to /usr/local/hadoop

```
dsh -a 'wget http://apache.mirrors.tds.net/hadoop/common/hadoop-2.7.7/hadoop-2.7.7.tar.gz'
dsh -a sudo tar zxvf hadoop-2.*
dsh -a 'rm -rf hadoop-2.*.tar.gz'
dsh -a 'sudo mv hadoop-2.*  /usr/local/hadoop'
dsh -a sudo chown -R `whoami`:`whoami` /usr/local/hadoop
```

## Step 5. Set Environment Variable on all nodes


#### 5.1. Edit bashrc file and past below contents at the end. (one per line)

##### Verify Java location
```
ls -lrt /usr/lib/jvm/java-8-openjdk-amd64
```

```
vi ~/.bashrc
```

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$JAVA_HOME/bin
export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop
export YARN_HOME=$HADOOP_HOME
```
##### Copy bashrc file to all nodes
```
scp ~/.bashrc snn:~
scp ~/.bashrc dn1:~
scp ~/.bashrc dn2:~
scp ~/.bashrc dn3:~
```

#### 5.2. Execute bash to apply the changes

```
dsh -a exec bash
exec bash
```

#### iii. Verify whether JAVA_HOME and HADOOP_HOME is copied to $PATH Variable

```
echo $PATH
```

## Step 6. Configure Hadoop

#### 6.1. Create directory for Namenode and datanode

```
mkdir -p /usr/local/hadoop/data/hdfs/namenode
dsh -a mkdir -p /usr/local/hadoop/data/hdfs/datanode
dsh -a sudo mkdir /var/log/hadoop/
dsh -a sudo chown `whoami`:`whoami` -R /var/log/hadoop
```

#### 6.2. Configure hadoop-env.sh

```
echo export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 >> /usr/local/hadoop/etc/hadoop/hadoop-env.sh
echo export HADOOP_LOG_DIR=/var/log/hadoop/ >> /usr/local/hadoop/etc/hadoop/hadoop-env.sh
```

##### Verify
```
tail /usr/local/hadoop/etc/hadoop/hadoop-env.sh
```

#### 6.3. Configure core-site.xml

`sudo vi /usr/local/hadoop/etc/hadoop/core-site.xml`

```
<property>
<name>fs.defaultFS</name>
<value>hdfs://nn.us-central1-a.c.lithe-grid-257118.internal:9000</value>
</property>
```

#### 6.4. Configure hdfs-site.xml

`sudo vi /usr/local/hadoop/etc/hadoop/hdfs-site.xml`

```
<property>
<name>dfs.replication</name>
<value>3</value>
</property>

<property>
<name>dfs.namenode.name.dir</name>
<value>file:///usr/local/hadoop/data/hdfs/namenode</value>
</property>

<property>
<name>dfs.datanode.name.dir</name>
<value>file:///usr/local/hadoop/data/hdfs/datanode</value>
</property>

<property>
<name>dfs.namenode.secondary.http-address</name>
<value>snn.us-central1-a.c.lithe-grid-257118.internal:50090</value>
</property>
```

### 6.5. Configure yarn-site.xml

`sudo vi /usr/local/hadoop/etc/hadoop/yarn-site.xml`

```
<property>
<name>yarn.nodemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>

<property>
<name>yarn.resourcemanager.hostname</name>
<value>nn.us-central1-a.c.lithe-grid-257118.internal</value>
</property>
```

#### 6.6. Configure mapred-site.xml

`cp /usr/local/hadoop/etc/hadoop/mapred-site.xml.template /usr/local/hadoop/etc/hadoop/mapred-site.xml`

`sudo vi /usr/local/hadoop/etc/hadoop/mapred-site.xml`

```
<property>
<name>mapreduce.framework.name</name>
<value>yarn</value>
</property>
```

#### 6.7. Configure Slaves file

Delete localhost entry from slaves file (or Comment it out) and update all slave hostnames, One per line.

`vi /usr/local/hadoop/etc/hadoop/slaves`

```
nn.us-central1-a.c.lithe-grid-257118.internal
snn.us-central1-a.c.lithe-grid-257118.internal
dn1.us-central1-a.c.lithe-grid-257118.internal
dn2.us-central1-a.c.lithe-grid-257118.internal
dn3.us-central1-a.c.lithe-grid-257118.internal
```

#### 6.8. Copy updated hadoop configuration files to all other nodes

```
cd /usr/local/hadoop/etc/hadoop
scp hadoop-env.sh core-site.xml  hdfs-site.xml mapred-site.xml yarn-site.xml slaves  snn:`pwd`
scp hadoop-env.sh core-site.xml hdfs-site.xml mapred-site.xml yarn-site.xml slaves   dn1:`pwd`
scp hadoop-env.sh core-site.xml hdfs-site.xml mapred-site.xml yarn-site.xml slaves   dn2:`pwd`
scp hadoop-env.sh core-site.xml hdfs-site.xml mapred-site.xml yarn-site.xml slaves   dn3:`pwd`
```

## Step 7. Format Namenode

```
hdfs namenode -format
```

> Look for Message ___Namenode has been successfully formatted___
> Verify data in namenode data directory

NOTE : 
__DO NOT FORMATE THE NAMENODE MULTIPLE TIME.__


## Step 8. Start Hadoop Services

#### Start HDFS Services:
```
start-dfs.sh
```

#### Start YARN Services:
```
start-yarn.sh
```

#### Start MR-JobHistory Server:
```
$HADOOP_HOME/sbin/mr-jobhistory-daemon.sh start historyserver
```

#### Check whether all services are up and running:

```
dsh -a "hostname;jps;echo "------------""
```

## Step 8. Verify WebUi

Service | Host URL | Default Port
------------- | -------------|-------------
Namenode | http://nn_host:port/ | 50070
ResourceManager | http://rm_host:port/ | 8088
MapReduce JobHistory Server | http://jhs_host:port/ | 19888
SecondaryNameNode | http://snn_host:port/ | 50090
DataNode | http://dn_host:port/ | 50075

## Step 9. Test Hadoop

### 9.1 Check HDFS commands
#### Run hdfs report

`hdfs dfsadmin -report`

#### Check Yarn status through command
`yarn node -list`

### 9.2 Run Sample MR Jobs

#### i) Pi
`hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar pi 10 100000`

#### ii) Word Count
##### Create File on local
`vi /tmp/word_example.txt`

##### Add some random text
```
The Apache Hadoop software library is a framework that allows for the distributed processing of
large data sets across clusters of computers using simple programming models. It is designed to
scale up from single servers to thousands of machines, each offering local computation and
storage. Rather than rely on hardware to deliver high-availability, the library itself is
designed to detect and handle failures at the application layer, so delivering a highly-
available service on top of a cluster of computers, each of which may be prone to failures.
Hadoop Common: The common utilities that support the other Hadoop modules.
Hadoop Distributed File System - HDFS: A distributed file system that provides high-throughput
access to application data.
Hadoop YARN: A framework for job scheduling and cluster resource management
Hadoop MapReduce: A YARN-based system for parallel processing of large data sets
Ambari: A web-based tool for provisioning, managing, and monitoring Apache Hadoop clusters which
includes support for Hadoop HDFS, Hadoop MapReduce, Hive, HCatalog, HBase, ZooKeeper, Oozie, Pig
and Sqoop. Ambari also provides a dashboard for viewing cluster health such as heatmaps and
ability to view MapReduce, Pig and Hive applications visually along with features to diagnose
their performance characteristics in a user-friendly manner
```

##### Create dir on hdfs and Load example file
```
hdfs dfs -mkdir –p /user/hdfs/sample_jobs/input
hdfs dfs -put /tmp/word_example.txt /user/hdfs/sample_jobs/input/
```

##### Verify whether file is uploaded
`hdfs dfs -cat /user/hdfs/sample_jobs/input/word*`

##### Run word count job
```
hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar wordcount
/user/hdfs/sample_jobs/input/ /user/hdfs/sample_jobs/output
```

##### Check output (CLI):
```
hdfs dfs -ls /user/hdfs/sample_jobs/output/
hdfs dfs -cat /user/hdfs/sample_jobs/output/part-*
```

##### Check output (Namenode WebUI)
1. Open Namenode WebUI http://nn_host:port/
2. Click “Utilities” --> “Browse the file System”
3. Navigate to the output directory (/user/hdfs/sample_jobs/output)
4. Click part-r-000* file
5. Download



