# Single Node Apache Hadoop - Cluster Deployement on Google Cloud Platform (GCP)

## Preparing the Environment (Pre-Requisites)

### 1) Configure Password-Less SSH (Perform this step only on node1)

1.1 - Generate ssh keys (Private & Public) using ssh-keygen command and press enter till it’s get complete for
default options

`ssh-keygen `

1.2 - Copy public key contents to authorized_keys

`cat id_rsa.pub >> authorized_keys `

1.3 - The logic behind password-less ssh is, (a) User public key (id_rsa.pub) should be available in "authorized_keys" (b) this "authorized_keys" must be available on all other nodes. 

For GCP follow below process which will automatically add "id_rsa.pub" to "authorized_keys" on all available nodes

`cat ~/.ssh/id_rsa.pub`

Copy public key contents and past it under SSH keys (GCP --> Compute Engine --> Metadata --> SSH Keys --> Edit --> Add new item)

1.4 - VERIFY:

You should be able to perform ssh to other nodes withoud password.

```
ssh localhost
ssh node2
exit
ssh node2
exit
ssh node3
```

### NOTE: Following steps needs to be performed on all nodes

### 2) Update hosts file

`sudo vi /etc/hosts`

### 3) Install & Enable NTP

```
sudo -i
yum install -y ntp
chkconfig ntpd on
```

### 4) Turn off iptables
RHEL6:

```
service iptables stop
chkconfig iptables on
```

RHEL7:

```
systemctl disable firewalld
service firewalld stop
```


### 5) Disable SELinux
```
sudo -i
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config && cat /etc/selinux/config
```

### 6) Configure Swappiness
```
sysctl vm.swappiness=10
echo 'vm.swappiness=10' >> /etc/sysctl.conf
```

### 7) Configure THP (transparent_hugepage)

```
echo 'never' > /sys/kernel/mm/transparent_hugepage/enabled
echo 'never' > /sys/kernel/mm/transparent_hugepage/defrag
```
### 8) Configure UMASK
Chek current umask
`umask`

Setting the umask for your current login session:

`umask 0022`

Permanently changing the umask for all interactive users:

`echo umask 0022 >> /etc/profile`

FYI:
A umask value of 022 grants read, write, execute permissions of 755 for new files or folders. Most Linux distros set 022 as the default umask value. If current umask value is 022 on your system, you can skip this step.

### 9) Install JAVA
```
sudo apt-get update

sudo apt-get install openjdk-7-jdk -y
```

## Download & Configure Hadoop

### 1) Download Hadoop tarball, untar it and move it to /usr/local/hadoop

```
wget https://archive.apache.org/dist/hadoop/core/hadoop-1.2.1/hadoop-1.2.1.tar.gz

sudo tar zxvf hadoop-1.*

sudo mv hadoop-1.2.1 /usr/local/hadoop

sudo chown -R `whoami`:`whoami` /usr/local/hadoop
```

Note:
Incase given package (hadoop-1.2.1) is not available on the link, use below link and download any hadoop 1.x version
https://archive.apache.org/dist/hadoop/core/

### 2) Setup ENVIRONMENT Variables for Hadoop
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

`start-dfs.sh`

5.2 - Start MR services

`start-mapred.sh`

### 6) Check what all services are running

`jps`

### 7) Verify through WebUI

| Component | URL Default | Port |
| --------- |-------------|------|
| Namenode | http://nn_host:port/ | 50070 |
| --------- | ------------| ------ |
| JobTracker | http://jt_host:port/ | 50030 |
|---------  |--------- |--------- |
| TaskTracker | http://tt_host:port/ | 50060 |
|--------- |--------- |--------- |
| SecondaryNameNode | http://snn_host:port/ | 50090 |
|--------- |--------- |--------- |
| DataNode | http://dn_host:port/ | 50075 |
|--------- |--------- |--------- |







