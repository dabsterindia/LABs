# Hortonworks Cluster Deployement - Using public/online repositories (i.e. Repos those are available on the Internet)


___Environment Details:___

Ambari | 2.6.2.2
------------- | -------------
HDP | 2.6.5
Operating System | Centos 6.x
Platform | Google Cloud Platform


Do not forget to check [Hortonworks Support Martrix](https://supportmatrix.hortonworks.com "Support Martrix") to check compatibility of your hardware/software.


------------------------------------------------------------------------------------------------------------------------------
## Step 1. Preparing the Environment (Prerequisites)

Follow [Hadoop Prerequisites](https://github.com/dabsterindia/LABs/blob/master/Cluster%20Deployments/Hadoop_Prerequisites.md "Hadoop Prerequisites") article to setup the environment for hadoop deployement

------------------------------------------------------------------------------------------------------------------------------
## Step 2. Download and Install Ambari Server

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

__Respond to the setup prompt:__ _Keep pressing 'ENTER' for default values_
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

```
ambari-server start
ambari-server status
```
_Open ambari UI (http://your-ambari.hostname.com:8080) on web browser and login using default credentials i.e. admin/admin


------------------------------------------------------------------------------------------------------------------------------
## Step 3. Deploy HDP Cluster

Follow below steps to deploy HDP cluster

### 3.1 : Launch the Ambari Cluster Install Wizard
To start creating a cluster, click __Launch Install Wizard__

### 3.2 : Name Your Cluster
Type a name for the cluster you want to create. 

For example:
"DevEnv",  "DabsterPOC", "Prod", "UAT", "DabsterDev01" , "DabUATEnv" etc

### 3.3 : Select Version
You can select HDP software version from the drop down & repo type from radio button

Do not change anything if you wanna deploye hdp with default options selected :)

### 3.4 : Install Options
#### a. Enter FQDN of each of your hosts. Example
```
edge01.hostname.com
master01.hostname.com
master02.hostname.com
slave01.hostname.com
slave02.hostname.com
slave03.hostname.com
```

OR
```
edge1.hostname.com
master[01-02].hostname.com
slave[01-25]
```

_You can get fqdn of your host with `hostname -f` command_

#### b. Host Registration Information
Copy & Past your user's private key (cat .ssh/id_rsa) from ambari-server host

_Make sure __"SSH User Account"__ field is updated with your password-less ssh username in case root is not configured for passwordless ssh.


### 3.5 : Confirm Hosts
At this step ambari check hosts to make sure they have the correct directories, packages, and processes required to continue the install.

Look at the __Warnings encountered__ (if any) and correct them

### 3.1 : Choose Services

Choose Services to install from all listed services as per your requirement


### 3.1 : Assign Masters
Ambari by default assign master services based on your hardware setup.

_Make sure you verify all service and choose appropriate host as per your need._


### 3.1 : Assign Slaves and Clients
Assign slave components (DataNodes, NodeManagers, and RegionServers) to appropriate hosts in your cluster.

_Do not install clients on all hosts_

### 3.1 : Customize Services
You are __strongly encouraged__ to review settings on this page. 

Ambari runs a python scripts to read filesystem configs and put values accordingly. You need to review all configs such as namenode & datanode data directory, log locations, memory & cpu configs etc

### 3.1 : Review
Review assignments & make sure everything is correct and go ahead for cluster __Deploy__'ment

### 3.1 : Install, Start and Test
Ambari will install all required packages on nodes.

Its fun to read ambari operation logs.

Go ahead and see operations to get more idea about __How does Ambari install hadoop services__

### 3.1 : Complete
Complete the HDP install wizard and explore Ambari Dashbord


#### Happy Learning !!!
