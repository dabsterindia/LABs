# How to use Ranger KMS

#### ___Goal: 
Our goal here is to: 
* Create HDFS encryption zone
* Upload data files in encryption zone 
* Allow users (eg: dab01)to access encrypted data


### Hadoop KMS commands used to manage keys and encryption zone
`$ hadoop key`

`$ hdfs crypto`


## Step 1. Login Ranger and Verify whether hdfs superuser has permissions on keys

#### 1.1 : Login Ranger with with keyadmin user credentials (default username/password is keyadmin/keyadmin)

![alt text](https://github.com/dabsterindia/LABs/blob/master/tmp/images/ranger_kms_keyadmin_login.png)

![alt text](https://github.com/dabsterindia/LABs/blob/master/tmp/images/ranger_kms_repo.png)

![alt text](https://github.com/dabsterindia/LABs/blob/master/tmp/images/ranger_kms_policies1.png)

#### 1.2 Verify whether hdfs (& hadoop superuser) has permissions on all keys
##### If hdfs user is not added in default kms policy, Edit default ranger kms policy & Add hdfs supperuser to default ranger kms policy and give `GET_METADATA` and `Generate_EEK` permissions.

## Step 2. Create Encryption Zone Key (ezkey)

#### 2.1: Create Key
Ranger KMS UI --> Encryption --> Key Manager --> Select Service (clustername_kms) --> Add New Key
```
Key Name : ezkey1
Cipher : AES/CTR/NoPadding
Length : 128
```
Save the changes

![alt text](https://github.com/dabsterindia/LABs/blob/master/tmp/images/ranger_kms_keymanager.png)

![alt text](https://github.com/dabsterindia/LABs/blob/master/tmp/images/ranger_kms_add_key.png)

#### 2.1: Allow permissions on key
Goto:
Access Manager --> kms repo (mycluster_kms) --> Add New Policy
Add new Ranger KMS policy for `ezkey1` and assign `Decrypt_EEK` permission to 'dab01' user 

## Step 3. - Create Encryption Zone on HDFS

#### 3.1: Create an empty directory on HDFS
```
$ su - hdfs
$ hdfs dfs -mkdir /ezone1
```

#### 3.1: Assign required permissions, owner, and group to encryption zone
```
$ su - hdfs
$ hdfs dfs -chown dab01:dabsters /ezone1
$ hdfs dfs -chmod 750 /ezone1
```

#### 3.2: Using the encryption zone key, designate the folder as an encryption zone.
```
$  su - hdfs
$  hdfs crypto -createZone -keyName ez-key -path /ezone1
$  hdfs crypto -listZones
```

#### 3.3: Try to create a file on hdfs using hdfs supperuser
```
su - hdfs
hdfs dfs -touchz /ezone1/testfile
```
_Command will fail because hdfs user does not have permission to create/write data in encryption zone (ezone1)

_ONLY dab01 is allowed to create/write files in ezone1

#### 3.4: Now Copy Files from/to an Encryption Zone using dab01 user who have permissions on ezkey1
```
su - dab01
echo "Welcome to Dasbter, We are learning Ranger KMS today" > /var/tmp/testfile.txt

hdfs dfs -copyFromLocal /var/tmp/testfile.txt  /ezone1
hdfs dfs -ls /ezone1
hdfs dfs -cat /ezone1/testfile.txt

hdfs dfs -touchz /ezone1/testfile
```

#### 3.5: Other users(including hdfs supperuser) can see files but cannot read/write files in encrypted zone
```
su - dab02
hdfs dfs -cat /ezone1/testfile.txt

su - hdfs
hdfs dfs -copytoLocal /ezone1/testfile.txt /tmp/
```


