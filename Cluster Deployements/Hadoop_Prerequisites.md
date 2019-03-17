# Preparing the Environment for Hadoop Deployement:

## 1. Configure Password-Less SSH (Perform this step only on node1)
     
   i. Generate ssh keys (Private & Public) using ssh-keygen command and press enter till it’s get complete for
default options

`ssh-keygen `

   ii. Copy public key contents to authorized_keys

`cat id_rsa.pub >> authorized_keys `

   iii. The logic behind password-less ssh is, (a) User public key (id_rsa.pub) should be available in "authorized_keys" (b) this "authorized_keys" must be available on all other nodes. 

For GCP follow below process which will automatically add "id_rsa.pub" to "authorized_keys" on all available nodes

`cat ~/.ssh/id_rsa.pub`

Copy public key contents and past it under SSH keys (GCP --> Compute Engine --> Metadata --> SSH Keys --> Edit --> Add new item)

   iv. VERIFY:

You should be able to perform ssh to other nodes withoud password.

```
ssh localhost
ssh node2
exit
ssh node2
exit
ssh node3
```

NOTE: Following steps needs to be performed on all nodes

## 2. Update hosts file

`sudo vi /etc/hosts`

## 3. Install & Enable NTP

```
sudo -i
yum install -y ntp
chkconfig ntpd on
```

## 4. Turn off iptables
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


## 5. Disable SELinux
```
sudo -i
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config && cat /etc/selinux/config
```

## 6. Configure Swappiness
```
sysctl vm.swappiness=10
echo 'vm.swappiness=10' >> /etc/sysctl.conf
```

## 7. Configure THP (transparent_hugepage)

```
echo 'never' > /sys/kernel/mm/transparent_hugepage/enabled
echo 'never' > /sys/kernel/mm/transparent_hugepage/defrag
```
## 8. Configure UMASK
Chek current umask
`umask`

Setting the umask for your current login session:

`umask 0022`

Permanently changing the umask for all interactive users:

`echo umask 0022 >> /etc/profile`

FYI:

A umask value of 022 grants read, write, execute permissions of 755 for new files or folders. Most Linux distros set 022 as the default umask value. If current umask value is 022 on your system, you can skip this step.
