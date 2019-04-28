# How to use Ranger KMS

## ___Goal: To create HDFS encryption zone for an end-user 'dab01'.__

### Hadoop KMS commands
`$ hadoop key`

`$ hdfs crypto`


## Step 1) - Login Ranger KMS UI 

#### Open Ranger UI and login with keyadmin user credentials (default username/password is keyadmin/keyadmin)


### a) Provide permissions on keys to hdfs supperuser

#### Edit default ranger kms policy & Add hdfs supperuser to default ranger kms policy and give `GET_METADATA` and `Generate_EEK` permissions.

### b) Create Encryption Zone Key

#### Ranger KMS UI --> Encryption --> Key Manager --> Select Service (clustername_kms) --> Add New Key 
```
Key Name : ezkey1
Cipher : AES/CTR/NoPadding
Length : 128
```
Save the changes

### c) Allow permissions on key
Add new Ranger KMS policy for `ezkey1` and assign `Decrypt_EEK` permission to 'dab01' user 

## Step 2) - Create Encryption Zone on HDFS

### a) Create an empty directory on HDFS
```
$ su - hdfs
$ hdfs dfs -mkdir /ezone1
```

### b) Assign required permissions, owner, and group to encryption zone
```
$ su - hdfs
$ hdfs dfs -chown dab01:dabsters /ezone1
$ hdfs dfs -chmod 750 /ezone1
```

### c) Using the new encryption zone key, designate the folder as an encryption zone.
```
$  su - hdfs
$  hdfs crypto -createZone -keyName ez-key -path /ezone1
$  hdfs crypto -listZones
```

### d) Try to create a file on hdfs using hdfs supperuser
```
su - hdfs
hdfs dfs -touchz /ezone1/testfile
```

### e) Now Copy Files from/to an Encryption Zone using dab01 user who have permissions on ezkey1
```
su - dab01
echo "Welcome to Dasbter, We are learning Ranger KMS today" > /var/tmp/testfile.txt

hdfs dfs -copyFromLocal /var/tmp/testfile.txt  /ezone1
hdfs dfs -ls /ezone1
hdfs dfs -cat /ezone1/testfile.txt

hdfs dfs -touchz /ezone1/testfile
```

### f) Other users(including hdfs supperuser) can see files but cannot read/write files in encrypted zone
```
su - dab02
hdfs dfs -cat /ezone1/testfile.txt

su - hdfs
hdfs dfs -copytoLocal /ezone1/testfile.txt /tmp/
```


