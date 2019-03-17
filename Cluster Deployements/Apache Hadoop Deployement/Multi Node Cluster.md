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
#### i. Install dsh

`sudo apt-get install –y dsh`

#### ii. Update machine.list file with all host entries. Comment out localhost.

`sudo vi /etc/dsh/machines.list`

```
#localhost
nn
snn
dn1
dn2
dn3
```

#### iii. Verify dsh

`dsh -a "hostname;echo "------------"`

## Step 3. Install JAVA 8 on all nodes

#### i. To add oracle Java repositories, we need to download python-software-properties. Install it using
below commands

`dsh -a sudo apt-get install -y python-software-properties debconf-utils`

#### ii. Add Oracle’s PPA(Personal Package Archive) to your list of sources so that Ubuntu knows where
to check for the updates.

`dsh -a sudo add-apt-repository -y ppa:webupd8team/java`

#### iii. Update repos and install java 8

```
dsh -a sudo apt-get update
dsh -a ";echo "oracle-java8-installer shared/accepted-oracle-license-v1-1 select true" | sudo debconf-set-selections"
```

`dsh -a sudo apt-get install -y oracle-java8-installer`









