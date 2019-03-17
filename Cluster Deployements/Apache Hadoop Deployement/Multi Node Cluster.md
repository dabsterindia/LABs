# Multi Node Apache Hadoop - Cluster Deployement on Google Cloud Platform (GCP)

___Environment Details:___

1 | 2
------------- | -------------
Operating System | Ubuntu 14.04 (For RHEL Follow [Hadoop Prerequisites](https://github.com/dabsterindia/LABs/blob/master/Cluster%20Deployements/Hadoop_Prerequisites.md "Hadoop Prerequisites") article)
Java Vesion | Java 8 Oracle
Hadoop Version | Hadoop-2.7.4

## Step 1. Preparing the Environment (Prerequisites)
#### FOR UBUNTU:
#### i. Configuer `/etc/hosts` file
#### ii. Configure passwordless ssh
#### iii. Verify firewall status, it must be disabled

#### For RHEL/Centos:
Follow [Hadoop Prerequisites](https://github.com/dabsterindia/LABs/blob/master/Cluster%20Deployements/Hadoop_Prerequisites.md "Hadoop Prerequisites") article to setup the environment for hadoop deployement


## Step 2. Install dsh and Update machine.list file
#### 2.1. Install dsh

`sudo apt-get install –y dsh`

#### 2.2. Update machine.list file with all host entries. Comment out localhost.

`sudo vi /etc/dsh/machines.list`

```
#localhost
nn
snn
dn1
dn2
dn3
```

#### 2.3. Verify dsh

`dsh -a "hostname;echo "------------"`

## Step 3. Install JAVA 8 on all nodes

#### 3.1. To add oracle Java repositories, we need to download python-software-properties. Install it using
below commands

`dsh -a sudo apt-get install -y python-software-properties debconf-utils`

#### 3.2. Add Oracle’s PPA(Personal Package Archive) to your list of sources so that Ubuntu knows where
to check for the updates.

`dsh -a sudo add-apt-repository -y ppa:webupd8team/java`

#### 3.3. Update repos and install java 8

```
dsh -a sudo apt-get update
dsh -a ";echo "oracle-java8-installer shared/accepted-oracle-license-v1-1 select true" | sudo debconf-set-selections"
```

`dsh -a sudo apt-get install -y oracle-java8-installer`

## Step 4. Install Hadoop:

#### Install Hadoop on all nodes using dsh utility, untar it and move to /usr/local/hadoop

```
dsh -a wget http://apache.mirrors.tds.net/hadoop/common/hadoop-2.7.4/hadoop-2.7.4.tar.gz
dsh -a sudo tar zxvf hadoop-2.*
dsh -a sudo mv hadoop-2.7.4 /usr/local/hadoop
dsh -a sudo chown -R `whoami`:`whoami` /usr/local/hadoop
```

## Step 5. Set Environment Variable on all nodes


#### 5.1. Edit bashrc file and past below contents at the end. (one per line)

`vi ~/.bashrc`

```
export JAVA_HOME=/usr/lib/jvm/java-8-oracle
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export PATH=$PATH:$JAVA_HOME/bin
export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop
export YARN_HOME=$HADOOP_HOME
```

#### 5.2. Execute bash to apply the changes

`dsh -a exec bash`

#### iii. Verify whether JAVA_HOME and HADOOP_HOME is copied to $PATH Variable

`echo $PATH`

## Step 6. Configure Hadoop

#### 6.1. Create directory for Namenode and datanode

```
dsh -a mkdir -p /usr/local/hadoop/data/hdfs/datanode
mkdir -p /usr/local/hadoop/data/hdfs/namenode
dsh -a sudo mkdir /var/log/hadoop/
dsh -a sudo chown `whoami`:`whoami` -R /var/log/hadoop
```

#### 6.2. Update Hadoop-env.sh

```
echo export JAVA_HOME=/usr/lib/jvm/java-8-oracle >> /usr/local/hadoop/etc/hadoop/hadoop-env.sh
echo export HADOOP_LOG_DIR=/var/log/hadoop/ >> /usr/local/hadoop/etc/hadoop/hadoop-env.sh
Update core-site.xml
```

#### 6.3. sudo vi /usr/local/hadoop/etc/hadoop/core-site.xml

```
<property>
<name>fs.defaultFS</name>
<value>hdfs://nn:9000</value>
</property>
```

#### 6.4. Update hdfs-site.xml

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
<value>snn:50090</value>
</property>
```

### 6.5. Update yarn-site.xml

`sudo vi /usr/local/hadoop/etc/hadoop/yarn-site.xml`

```
<property>
<name>yarn.nodemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>

<property>
<name>yarn.resourcemanager.hostname</name>
<value>nn</value>
</property>
```

#### 6.6. Update mapred-site.xml

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
#localhost
dn1
dn2
dn3
```

#### 6.8. Copy Hadoop conf files to other nodes

```
cd /usr/local/hadoop/etc/hadoop &amp;&amp; scp hadoop-env.sh core-site.xml hdfs-site.xml mapred-site.xml yarn-site.xml snn:`pwd`
cd /usr/local/hadoop/etc/hadoop &amp;&amp; scp hadoop-env.sh core-site.xml hdfs-site.xml mapred-site.xml yarn-site.xml dn1:`pwd`
cd /usr/local/hadoop/etc/hadoop &amp;&amp; scp hadoop-env.sh core-site.xml hdfs-site.xml mapred-site.xml yarn-site.xml dn2:`pwd`
cd /usr/local/hadoop/etc/hadoop &amp;&amp; scp hadoop-env.sh core-site.xml hdfs-site.xml mapred-site.xml yarn-site.xml dn3:`pwd`
```

## Step 7. Format Namenode

`hdfs namenode -format`

## Step 8. Start Services
#### Start HDFS Services:
`start-dfs.sh`

#### Start YARN Services:
`start-yarn.sh`

#### Start MR- JobHistory Server:
`$HADOOP_HOME/sbin/mr-jobhistory-daemon.sh start historyserver`

Check whether all services are up and running:

`dsh -a "hostname;jps;echo "------------""`

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
Run hdfs report

`hdfs dfsadmin -report`

Check Yarn status through command
`yarn node -list`

### 9.2 Run Sample MR Jobs

#### i) Pi
hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar pi 10 100000

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



