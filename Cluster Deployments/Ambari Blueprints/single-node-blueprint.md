# Create single node HDP cluster using Ambari Blueprints


___Environment Details:___

Column 1 | Column 2
------------- | -------------
Operating System | CentOS Linux release 7.6.1810
HDP Stack Version | HDP-3.1
Ambari Version | Ambari-2.7.3
Installed Services | HDFS, YARN, MapReduce2, ZooKeeper
Hostname -f | hdp33.europe-west6-a.c.silent-base-227511.internal


## Step 0. Prerequisites
```
sudo -i
systemctl disable firewalld
service firewalld stop

setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config && cat /etc/selinux/config

```

## Step 1. Prepare Ambari Server and Agents

#### 1.1. Download ambari repository
```
# sudo -i

# wget -nv http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.7.3.0/ambari.repo -O /etc/yum.repos.d/ambari.repo
```

#### 1.2. Install & setup Ambari server
```
# yum install ambari-server -y
# ambari-server setup -s
# ambari-server start
```

#### 1.3. Install Ambari agent and set ambari-server hostname in ambari.ini file

```
# yum install ambari-agent -y
# ambari-agent reset $(hostname -f)
# ambari-agent start

# cat /etc/ambari-agent/conf/ambari-agent.ini | grep -i 'hostname' -B1
```


#### 1.3. Confirm whether ambari-agent hosts are registered with the Server

`#  curl -u admin:admin http://$(hostname -f):8080/api/v1/hosts`


## Step 2. Prepare & deploy cluster
### Create Two JSON Files (1) cluster_configuration.json & (2) hostmapping.json

#### 2.1. cluster_configuration.json contains services to be installed & service configurations

`#  vi cluster_configuration.json`

```
{
  "host_groups" : [
    {
      "name" : "my_host_group",     
      "components" : [
        {
          "name" : "NAMENODE"
        },
        {
          "name" : "SECONDARY_NAMENODE"
        },       
        {
          "name" : "DATANODE"
        },
        {
          "name" : "HDFS_CLIENT"
        },
        {
          "name" : "RESOURCEMANAGER"
        },
        {
          "name" : "NODEMANAGER"
        },
        {
          "name" : "YARN_CLIENT"
        },
        {
          "name" : "HISTORYSERVER"
        },
        {
          "name" : "APP_TIMELINE_SERVER"
        },
        {
          "name" : "MAPREDUCE2_CLIENT"
        },
        {
          "name" : "ZOOKEEPER_SERVER"
        },
        {
          "name" : "ZOOKEEPER_CLIENT"
        }
      ],
      "cardinality" : "1"
    }
  ],
  "Blueprints" : {
    "blueprint_name" : "single-node-dabster-inc",
    "stack_name" : "HDP",
    "stack_version" : "3.1"
  }
}
```


#### Step 2.2. Register Blueprint with ambari-server

```
# curl -u admin:admin -i -H 'X-Requested-By: ambari' -X POST -d @cluster_configuration.json http://$(hostname -f):8080/api/v1/blueprints/single-node-dabster-inc

```

#### 2.3. hostmapping.json file contains host details of your cluster

`#  vi hostmapping.json`

```
{
  "blueprint" : "single-node-dabster-inc",
  "host_groups" :[
    {
      "name" : "my_host_group", 
      "hosts" : [         
        {
          "fqdn" : "hdp33.europe-west6-a.c.silent-base-227511.internal"
        }
      ]
    }
  ]
}

```
_Don't forget to change the hostname :)_

#### Step 2.4. Post the cluster to the Ambari Server to provision the cluster (Create HDP Cluster)

```
#  curl -H "X-Requested-By: ambari" -X POST -u admin:admin http://$(hostname -f):8080/api/v1/clusters/clustername -d @hostmapping.json

```

## Step 3. Login Ambari UI

`http://<ambari-server>:8080/`
  

##### Optionally You can Check the status of cluster installation through Amabri API as follows

```
# curl -H "X-Requested-By: ambari" -X GET -u admin:admin http://$(hostname -f):8080/api/v1/clusters/clustername/requests/

# curl -H "X-Requested-By: ambari" -X GET -u admin:admin http://$(hostname -f):8080/api/v1/clusters/clustername/requests/<request_number>
```






