# Single Node Apache Hadoop - Cluster Deployement on Google Cloud Platform (GCP)

## Preparing the Environment (Pre-Requisites)

### 1) Configure Password-Less SSH

1.1 - Generate ssh keys (Private & Public) using ssh-keygen command and press enter till it’s get complete for
default options

` $ ssh-keygen `

1.2 - Copy public key contents to authorized_keys
` $ cat id_rsa.pub >> authorized_keys `

1.3 - This "authorized_keys" should be available on all other nodes. For GCP follow below process which will automatically add "id_rsa.pub" to "authorized_keys" on all available nodes

`$ cat ~/.ssh/id_rsa.pub`

Copy public key and past under ssh keys (GCP --> Compute Engine --> Metadata --> SSH Keys --> Edit --> Add new item)

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


### 2) Update hosts file

`# sudo vi /etc/hosts`

### 2) Turn off iptables
```
# sudo iptables stop
# sudo chkconfig iptables on
```

### 3) 
