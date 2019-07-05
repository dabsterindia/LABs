# Ansible 
### Setup passwordless ssh

```
yum install ansible -y

mkdir setup_cluster
```

`vi hosts`

```
[all]
node1 ansible_user=dab01
node2 ansible_user=dab01
node3 ansible_user=dab01
node4 ansible_user=dab01
```

### Test
```
ansible all -i hosts -m command -a "hostname -f"
```
