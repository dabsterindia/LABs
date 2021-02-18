# Single Node Apache Hadoop - Cluster Deployment on Google Cloud Platform (GCP)

___Environment Details:___

1 | 2
------------- | -------------
Operating System | Ubuntu
Java Vesion | Java - OpenJDK 7
Hadoop Version | Hadoop-1.2.1


## Step 1. Preparing the Environment (Prerequisites)

Follow [Hadoop Prerequisites](https://github.com/dabsterindia/LABs/blob/master/Cluster%20Deployments/Hadoop_Prerequisites.md "Hadoop Prerequisites") article to setup the environment for hadoop deployement

OR

```
ssh-keygen
```
```
cat id_rsa.pub >> authorized_keys 
```
```
ssh localhost
```

## Step 2. Install JAVA
```
sudo add-apt-repository ppa:openjdk-r/ppa  
sudo apt-get update   
sudo apt-get install openjdk-7-jdk 
```

## Step 3. Download & Configure Hadoop

#### 3.1. Download Hadoop tarball, untar it and move it to /usr/local/hadoop

```
wget https://archive.apache.org/dist/hadoop/core/hadoop-1.2.1/hadoop-1.2.1.tar.gz

sudo tar zxvf hadoop-1.*

sudo mv hadoop-1.2.1 /usr/local/hadoop

sudo chown -R `whoami`:`whoami` /usr/local/hadoop
```

Note:
Incase given package (hadoop-1.2.1) is not available on the link, use below link and download any hadoop 1.x version
https://archive.apache.org/dist/hadoop/core/

#### 3.2. Setup ENVIRONMENT Variables for Hadoop
Edit bashrc file and past below contents at the end. (one per line):

```
sudo vi ~/.bashrc
````

```
export HADOOP_PREFIX=/usr/local/hadoop
export PATH=$PATH:$HADOOP_PREFIX/bin
export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
export PATH=$PATH:$JAVA_HOME
```

Execute bash to apply the changes:

`exec bash`

Verify:
```
echo $PATH

jps
```

## 4. Configure Hadoop files
#### 4.1 - Create Directories for Namenode metadata

```
mkdir -p /usr/local/hadoop/data/hdfs/namenode
chown -R `whoami`:`whoami` /usr/local/hadoop/data/hdfs/namenode
```

#### 4.2 - Configure hadoop-env.sh

`vi /usr/local/hadoop/conf/hadoop-env.sh`

```
export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
export HADOOP_OPTS=-Djava.net.preferIPV4Stack=true
```

#### 4.3 - Configure core-site.xml

```
vi /usr/local/hadoop/conf/core-site.xml
```
```
<property>
<name>fs.default.name</name>
<value>hdfs://node1.example.com:9000</value>
</property>

<property>
<name>hadoop.tmp.dir</name>
<value>/usr/local/hadoop/data/hdfs/namenode</value>
</property>
```
Note: replace "node1.example.com" with your host fqdn

#### 4.3 - Configure hdfs-site.xml

`vi /usr/local/hadoop/conf/hdfs-site.xml`

```
<property>
<name>dfs.replication</name>
<value>1</value>
</property>
```

#### 4.4 - Configure mapred-site.xml

`vi /usr/local/hadoop/conf/mapred-site.xml`

```
<property>
<name>mapred.job.tracker</name>
<value>node1.example.com:9001</value>
</property>
```
Note: replace "node1.example.com" with your host fqdn

## 5. Formate Namenode

`hadoop namenode -format`

> Look for Message ___Namenode has been successfully formatted___

> Verify data in namenode data directory

NOTE : 
__DO NOT FORMATE THE NAMENODE MULTIPLE TIME.__

## 6. Start Services

#### 6.1 - Start HDFS Services:

`start-dfs.sh`

#### 6.2 - Start MR services

`start-mapred.sh`

## 7. Check what all services are running

`jps`

## 8. Verify through WebUI

| Component | URL Default | Port |
| --------- |-------------|------|
| Namenode | http://nn_host:port/ | 50070 |
| JobTracker | http://jt_host:port/ | 50030 |
| TaskTracker | http://tt_host:port/ | 50060 |
| SecondaryNameNode | http://snn_host:port/ | 50090 |
| DataNode | http://dn_host:port/ | 50075 |

## 9. Test HDFS

#### i) Run “ls” on hdfs

`hadoop fs -ls /`

#### ii) Create a directory

`hadoop fs -mkdir /test`

#### iii) Run hdfs report to get the detailed information about the Hadoop filesystem

`hadoop dfsadmin -report`

#### iv) Check all Hadoop filesystem commands

`hadoop fs`

#### v) Check all hdfs admin commands
`hadoop dfsadmin`


## 10. TEST MapReduce
### Run Sample MapReduce Jobs

#### i) PI

`hadoop jar /usr/local/hadoop/hadoop-examples-*.jar pi 10 100000` 


#### ii) Word Count
##### Create File on local

`vi /tmp/word_example.txt`


#####  Add some random text
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
hadoop fs -mkdir –p /user/hdfs/sample_jobs/input
hadoop fs -put /tmp/word_example.txt /user/hdfs/sample_jobs/input/
```

##### Verify whether file is uploaded

`hadoop fs -cat /user/hdfs/sample_jobs/input/word*`

##### Run word count job

`hadoop jar /usr/local/hadoop/hadoop-examples-*.jar wordcount /user/hdfs/sample_jobs/input/
/user/hdfs/sample_jobs/output`

___You can monitor the status of this mapreduce jobs on JobTracker Web Interface___ (http://jt_host:port )

##### Check output (CLI):

```
hadoop fs -ls /user/hdfs/sample_jobs/output/
hadoop fs -cat /user/hdfs/sample_jobs/output/part-*
```


##### Check output (Namenode WebUI)

a. Open Namenode WebUI http://nn_host:port/

b. Click “Utilities” &gt; “Browse the file System”

c. Navigate to the output directory (/user/hdfs/sample_jobs/output)

d. Click part-r-000* file


