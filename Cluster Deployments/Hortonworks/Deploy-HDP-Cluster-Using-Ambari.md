# Hortonworks Cluster Deployement - Using public/online repositories (i.e. Repos those are available on the Internet)


___Environment Details:___

1 | 2
------------- | -------------
Operating System | Centos 6.x
Ambari | 2.6.2.2
HDP | 2.6.5
Platform | Google Cloud Platform


Do not forget to check [Hortonworks Support Martrix](https://supportmatrix.hortonworks.com "Support Martrix") to check compatibility of your hardware/software.


------------------------------------------------------------------------------------------------------------------------------
## Step 1. Preparing the Environment (Prerequisites)
------------------------------------------------------------------------------------------------------------------------------
Follow [Hadoop Prerequisites](https://github.com/dabsterindia/LABs/blob/master/Cluster%20Deployments/Hadoop_Prerequisites.md "Hadoop Prerequisites") article to setup the environment for hadoop deployement

------------------------------------------------------------------------------------------------------------------------------
## Step 2. Download and Install Ambari Server
------------------------------------------------------------------------------------------------------------------------------
### 2.1 : Download the Ambari Repository
```sudo -i
wget -nv http://public-repo-1.hortonworks.com/ambari/centos6/2.x/updates/2.6.2.2/ambari.repo -O /etc/yum.repos.d/ambari.repo
```

`cat /etc/yum.repos.d/ambari.repo`

### 2.2 : Install the Ambari Server

```
yum clean all
yum repolist
yum install ambari-server -y
```

### 2.3 : Set Up the Ambari Server

`ambari-server setup`

Respond to the setup prompt: _Keep pressing 'ENTER' for default values_
```
Customize user account for ambari-server daemon [y/n] (n)? n

[1] Oracle JDK 1.8 + Java Cryptography Extension (JCE) Policy Files 8
[2] Oracle JDK 1.7 + Java Cryptography Extension (JCE) Policy Files 7
[3] Custom JDK
==============================================================================
Enter choice (1):1

Enable Ambari Server to download and install GPL Licensed LZO packages [y/n] (n)?n

Enter advanced database configuration [y/n] (n)?n

Ambari Server 'setup' completed successfully.
```

### 2.4 : Start Ambari Server

`ambari-server start`
_Open web browser and logint

------------------------------------------------------------------------------------------------------------------------------
## Step 3. Deploy HDP Cluster
------------------------------------------------------------------------------------------------------------------------------

