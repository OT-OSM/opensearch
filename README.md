## Ansible Role: Opensearch

Ansible role to configure Opensearch standalone and cluster integration

Some of the highlighting features are:-

  - Standalone setup of Opensearch
  - Cluster setup of Opensearch
  - Usermanagement 
 
### Supported OS

Ubuntu:
  - focal
  - bionic
Centos
  - 7
  - 8
AmaozonLinux:2

### Requirements

**Java**

### Roles Variables

#### Mandatory Variables

|**Variable**|**Default Value**|**Possible Values**|**Description**|
|------------|-----------------|-------------------|---------------|
| Standalone | true | <ul><li>true</li><li>false</li></ul> | boolean | Setup opensearch standalone if this value is true then the value of cluster should be false|
| Cluster | false | <ul><li>true</li><li>false</li></ul> | boolean | Setup opensearch Cluster if this value is true then the value of standalone should be false |
| admin_password | Strong@321 | *Password* | string | Password for opensearch username |



#### Optional Variables

|**Variable**|**Default Value**|**Possible Values**|**Description**|
|------------|-----------------|-------------------|---------------|
| Version | 1.3.0 | *opensearch Version* | string | Default Version of opensearch |
| VmMaxMapCountValue | 262144 | Count value | String | Setting vm.max_map_count in opensearch nodes |
| FsFileMaxValue | 65536 | Count value | String | Setting file max value on opensearch nodes |
| Min_HeapSize | 2 | Count value | String | Setting minimum heap size|
| Max_HeapSize | 2 | Count value | String | Setting maximum heap size|
| clusterName | my-application | Application name | String | You can use any application name|
| OpenSearchPort | 9200 | Port number | String | can use port no as per the architecture|
| LocalSystemDest | /tmp/opensearch-nodecerts | Path of local system | String | Certificates are generated in local system so have to give a path to create certificate|



### Usage

The inventory for Opensearch role should look like this:-

```ini
[opensearch]
opensearchnode1 ansible_ssh_host=3.18.112.95 node_type=master,data 
opensearchnode2 ansible_ssh_host=18.222.251.138 node_type=master,data 
opensearchnode3 ansible_ssh_host=18.220.251.141 node_type=master,data 

[opensearch:vars]
ansible_ssh_user=ubuntu
ansible_ssh_private_key_file= 

[cluster_nodes]
opensearchnode1
opensearchnode2
opensearchnode3

[cluster_master_nodes]
opensearchnode1
opensearchnode2
opensearchnode3
```


An example playbook should look like this:-

```yaml
---
- name: opensearch
  hosts: all
  gather_facts: true
  roles:
   - { role: opensearch }
```

and for running the ansible role, we will use ansible cli.

```shell
ansible-playbook -i tests/inventory tests/test.yml
```
## Authors

**[Varghese Kurian](varghese.palamoottil@opstree.com)**
