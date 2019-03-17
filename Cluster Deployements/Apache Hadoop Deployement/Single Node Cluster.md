# Single Node Apache Hadoop - Cluster Deployement on Google Cloud Platform (GCP)

## Step 1. Preparing the Environment (Prerequisites)

Follow [Named Link](https://github.com/dabsterindia/LABs/blob/master/Cluster%20Deployements/Hadoop_Prerequisites.md "Hadoop Prerequisites") article to setup the environment for hadoop deployement

## Step 2. Install JAVA
```
sudo apt-get update

sudo apt-get install openjdk-7-jdk -y
```

## Step 3. Download & Configure Hadoop

#### i. Download Hadoop tarball, untar it and move it to /usr/local/hadoop

```
wget https://archive.apache.org/dist/hadoop/core/hadoop-1.2.1/hadoop-1.2.1.tar.gz

sudo tar zxvf hadoop-1.*

sudo mv hadoop-1.2.1 /usr/local/hadoop

sudo chown -R `whoami`:`whoami` /usr/local/hadoop
```

Note:
Incase given package (hadoop-1.2.1) is not available on the link, use below link and download any hadoop 1.x version
https://archive.apache.org/dist/hadoop/core/

#### ii. Setup ENVIRONMENT Variables for Hadoop
Edit bashrc file and past below contents at the end. (one per line):

`vi ~/.bashrc`

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

### 3) Configure Hadoop files
3.1 - Create Directories for Namenode metadata

```
mkdir -p /usr/local/hadoop/data/hdfs/namenode
chown -R `whoami`:`whoami` /usr/local/hadoop/data/hdfs/namenode
```

3.2 - Configure hadoop-env.sh

`vi /usr/local/hadoop/conf/hadoop-env.sh`

```
export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
export HADOOP_OPTS=-Djava.net.preferIPV4Stack=true
```

3.3 - Configure core-site.xml

`vi /usr/local/hadoop/conf/core-site.xml`

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

3.3 - Configure hdfs-site.xml

`vi /usr/local/hadoop/conf/hdfs-site.xml`

```
<property>
<name>dfs.replication</name>
<value>1</value>
</property>
```

3.4 - Configure mapred-site.xml

`vi /usr/local/hadoop/conf/mapred-site.xml`

```
<property>
<name>mapred.job.tracker</name>
<value>master.example.com:9001</value>
</property>
```

#### 4) Formate Namenode

`hadoop namenode -format`

> Look for Message ** * Namenode has been successfully formatted *  **

> Verify data in namenode data directory

### 5) Start Services

5.1 - Start HDFS Services:

```start-dfs.sh```

5.2 - Start MR services

```start-mapred.sh```

### 6) Check what all services are running

```jps```

### 7) Verify through WebUI

| Component | URL Default | Port |
| --------- |-------------|------|
| Namenode | http://nn_host:port/ | 50070 |
| JobTracker | http://jt_host:port/ | 50030 |
| TaskTracker | http://tt_host:port/ | 50060 |
| SecondaryNameNode | http://snn_host:port/ | 50090 |
| DataNode | http://dn_host:port/ | 50075 |

### 8) Test HDFS

### 9) TEST MapReduce





