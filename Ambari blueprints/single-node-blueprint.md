# Create single node HDP cluster using Ambari Blueprints

Note:
I have used Ambari-2.7.3 & HDP-3.1. Kinldy change respective repos to deploy different versions as per your requirement

## Step 0. Pre-requisites
```
sudo -i
systemctl disable firewalld
service firewalld stop

setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config && cat /etc/selinux/config

```

## Step 1. Prepare Ambari Server and Agents

```
sudo -i

wget -nv http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.7.3.0/ambari.repo -O /etc/yum.repos.d/ambari.repo

yum install ambari-server -y
ambari-server setup
ambari-server start


yum install ambari-agent -y
ambari-agent reset $(hostname -f)
ambari-agent start

cat /etc/ambari-agent/conf/ambari-agent.ini | grep -i 'hostname' -B1
```

## Step 2. Create Two Files as follows and add the hosts and hdp services and their components in it.

vim cluster_configuration.json
vim hostmapping.json


## Step 3. Register the Blueprint with ambari with following command

```
curl -u admin:admin -i -H 'X-Requested-By: ambari' -X POST -d @cluster_configuration.json http://$(hostname -f):8080/api/v1/blueprints/nameofblueprint

```
