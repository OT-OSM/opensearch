# OpenSearch Ansible Role (Multi-OS + HA Cluster)

This Ansible role installs and configures an **OpenSearch cluster** with support for **multiple Linux distributions** and **High Availability (HA)** via multi-node clustering.

It also supports optional installation of **OpenSearch Dashboards**, and provides toggles for enabling/disabling:
- **TLS**
- **Security Plugin Initialization**

>  Tested using a 3-node cluster (Ubuntu + Amazon Linux + RHEL-family mix)

---

## Table of Contents

1. [Features](#features)  
2. [Supported Operating Systems](#supported-operating-systems)  
3. [Architecture Overview](#architecture-overview)  
4. [Pre-requisites](#pre-requisites)  
5. [Role Directory Structure](#role-directory-structure)  
6. [How to Use](#how-to-use)  
7. [Inventory Example](#inventory-example)  
8. [Variables (Configuration Reference)](#variables-configuration-reference)  
9. [Sample Variable Files](#sample-variable-files)  
10. [TLS & Security Options](#tls--security-options)  
11. [Run the Role](#run-the-role)  
12. [Validation & Health Checks](#validation--health-checks)  
13. [High Availability (HA) Testing](#high-availability-ha-testing)  
14. [Dashboards Access](#dashboards-access)  
15. [Troubleshooting](#troubleshooting)  
16. [Security Best Practices](#security-best-practices)  

---

## Features

✅ Multi-OS OpenSearch installation (Debian + RPM systems)  
✅ OpenSearch cluster setup with multi-node discovery  
✅ High Availability with replicas + node recovery  
✅ System tuning:
- disables swap (optional)
- sets `vm.max_map_count`
- configures file descriptor limits & systemd overrides  
✅ Supports **OpenSearch Dashboards** (optional)  
✅ Supports TLS and security plugin initialization (optional)  
✅ Tags supported: `install`, `packages`, `java`, `system`, `security`, `dashboards`

---

## Supported Operating Systems

This role supports:

| OS Family | Examples |
|----------|----------|
| Debian   | Ubuntu 20.04/22.04, Debian 11/12 |
| RedHat   | RHEL 8/9, CentOS, Rocky Linux |
| Amazon   | Amazon Linux 2 / Amazon Linux 2023 |
| SUSE     | openSUSE / SLES (rpm) |

>  Note: RPM systems may require GPG/signature handling depending on OS crypto policies.

---

## Architecture Overview

The OpenSearch cluster is formed as:

- **Cluster → Nodes → Index → Shards/Replicas**
- Cluster uses:
  - `discovery.seed_hosts`
  - `cluster.initial_cluster_manager_nodes`

Each node can run:
- cluster_manager role  
- data role  
- ingest role  

Example of a 3-node HA cluster:
- `os-a` (cluster_manager + data + ingest)
- `os-b` (cluster_manager + data + ingest)
- `os-c` (cluster_manager + data + ingest)

---

## Pre-requisites

- Ansible 2.15+ recommended  
- SSH access to all nodes  
- Ports allowed in Security Groups / Firewall:

| Service | Port |
|--------|------|
| OpenSearch HTTP | 9200 |
| OpenSearch Transport | 9300 |
| Dashboards (optional) | 5601 |

---

## Role Directory Structure

Example structure:

```

opensearch/
├── tasks/
│   ├── main.yml
│   ├── prereqs.yml
│   ├── install.yml
│   ├── configure.yml
│   ├── security.yml
│   ├── dashboards.yml
│   └── validate.yml
├── templates/
│   ├── opensearch.yml.j2
│   └── dashboards.yml.j2
├── handlers/
│   └── main.yml
├── defaults/
│   └── main.yml
├── vars/
│   └── main.yml
├── files/
│   └── security/
│       ├── internal_users.yml
│       ├── roles.yml
│       ├── roles_mapping.yml
│       └── ...
└── README.md

````

---

## How to Use

1. Create your inventory  
2. Add variables in `group_vars/`  
3. Run the playbook  

---

## Inventory Example

`inventory.ini`

```ini
[opensearch]
os-a ansible_host=<PUBLIC_IP_1> ansible_user=ec2-user
os-b ansible_host=<PUBLIC_IP_2> ansible_user=ubuntu
os-c ansible_host=<PUBLIC_IP_3> ansible_user=ec2-user

[opensearch:vars]
ansible_ssh_private_key_file=./Oskey.pem
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
````

>  Ensure correct `ansible_user` per OS (Ubuntu = `ubuntu`, Amazon/RHEL = `ec2-user`)

---

## Variables (Configuration Reference)

### Cluster Basics

| Variable                    | Example           | Description        |
| --------------------------- | ----------------- | ------------------ |
| `opensearch_cluster_name`   | `opensearch-prod` | Cluster name       |
| `opensearch_version`        | `2.11.1`          | OpenSearch version |
| `opensearch_network_host`   | `0.0.0.0`         | Bind host          |
| `opensearch_http_port`      | `9200`            | HTTP Port          |
| `opensearch_transport_port` | `9300`            | Transport Port     |

---

### Node Discovery

| Variable                                   | Example                                       |
| ------------------------------------------ | --------------------------------------------- |
| `opensearch_seed_hosts`                    | `["172.31.0.10","172.31.0.11","172.31.0.12"]` |
| `opensearch_initial_cluster_manager_nodes` | `["os-a","os-b","os-c"]`                      |

---

### System Tuning

| Variable                      | Example  | Description             |
| ----------------------------- | -------- | ----------------------- |
| `opensearch_disable_swap`     | `true`   | Disables swap           |
| `opensearch_vm_max_map_count` | `262144` | Required for OpenSearch |
| `opensearch_file_descriptors` | `65535`  | File descriptor limit   |

---

### Dashboards

| Variable                        | Example      | Description               |
| ------------------------------- | ------------ | ------------------------- |
| `opensearch_dashboards_enabled` | `true/false` | Install dashboards or not |
| `opensearch_dashboards_node`    | `os-a`       | Node where dashboards run |

---

## Sample Variable Files

###  `group_vars/opensearch/main.yml` (Non-TLS / Non-Security)

Use this for lab/testing where security is not needed.

```yaml
# OpenSearch Basics
opensearch_cluster_name: "opensearch-prod"
opensearch_version: "2.11.1"

# Paths
opensearch_home: "/usr/share/opensearch"
opensearch_config_dir: "/etc/opensearch"
opensearch_data_dir: "/var/lib/opensearch"
opensearch_log_dir: "/var/log/opensearch"
opensearch_pid_dir: "/var/run/opensearch"

# Networking
opensearch_network_host: "0.0.0.0"
opensearch_http_port: 9200
opensearch_transport_port: 9300

# Cluster discovery (use private IPs)
opensearch_seed_hosts:
  - "172.31.45.95"
  - "172.31.23.201"
  - "172.31.46.133"

opensearch_initial_cluster_manager_nodes:
  - "os-a"
  - "os-b"
  - "os-c"

# Node roles
opensearch_node_roles:
  - cluster_manager
  - data
  - ingest

# System tuning
opensearch_disable_swap: true
opensearch_vm_max_map_count: 262144
opensearch_file_descriptors: 65535
opensearch_max_locked_memory: -1

# Dashboards (optional)
opensearch_dashboards_enabled: true
opensearch_dashboards_node: "os-a"

# TLS/Security toggles
opensearch_tls_enabled: false
opensearch_security_enabled: false
```

---

###  `group_vars/opensearch/vault.yml` (Vault Passwords)

Store sensitive values here and encrypt using ansible-vault.

Example:

```yaml
vault_opensearch_admin_password: "ChangeMe@123"
```

Encrypt:

```bash
ansible-vault encrypt group_vars/opensearch/vault.yml
```

Run:

```bash
ansible-playbook -i inventory.ini site.yml --ask-vault-pass
```

---

## TLS & Security Options

This role supports 2 major modes.

---

### TLS OFF + Security OFF (Default)

 Easy for learning/testing
 Works quickly on 3-node cluster
 Not recommended for production

```yaml
opensearch_tls_enabled: false
opensearch_security_enabled: false
```

---

### TLS ON + Security ON (Recommended for Production)

 Encrypted traffic
 Authentication + RBAC
 Security plugin initialization possible (securityadmin)

#### Recommended variables (example)

```yaml
opensearch_tls_enabled: true
opensearch_security_enabled: true

# Certificates location (example)
opensearch_tls_cert_dir: "/etc/opensearch/certs"

opensearch_tls_ca_cert: "{{ opensearch_tls_cert_dir }}/root-ca.pem"
opensearch_tls_node_cert: "{{ opensearch_tls_cert_dir }}/node.pem"
opensearch_tls_node_key: "{{ opensearch_tls_cert_dir }}/node-key.pem"
opensearch_tls_admin_cert: "{{ opensearch_tls_cert_dir }}/admin.pem"
opensearch_tls_admin_key: "{{ opensearch_tls_cert_dir }}/admin-key.pem"

# Security users
opensearch_admin_username: "admin"
opensearch_admin_password: "{{ vault_opensearch_admin_password }}"
```

---

## Security Admin Initialization (securityadmin.sh)

When `opensearch_security_enabled: true`, OpenSearch Security plugin requires initialization of security index using:

 `securityadmin.sh`

Example command (for reference):

```bash
/usr/share/opensearch/plugins/opensearch-security/tools/securityadmin.sh \
  -cd /usr/share/opensearch/plugins/opensearch-security/securityconfig/ \
  -icl -nhnv \
  -cacert /etc/opensearch/certs/root-ca.pem \
  -cert /etc/opensearch/certs/admin.pem \
  -key /etc/opensearch/certs/admin-key.pem
```

> If TLS is OFF, security admin may run in non-TLS mode (not recommended for production).

---

## Run the Role

Example playbook:

`site.yml`

```yaml
- name: Install & Configure OpenSearch Cluster with HA
  hosts: opensearch
  become: true
  roles:
    - opensearch
```

Run:

```bash
ansible-playbook -i inventory.ini site.yml --ask-vault-pass
```

---

## Validation & Health Checks

### 1) Check OpenSearch version on all nodes

```bash
ansible -i inventory.ini opensearch -b -m shell -a "/usr/share/opensearch/bin/opensearch --version" --ask-vault-pass
```

---

### 2) Check service status on all nodes

```bash
ansible -i inventory.ini opensearch -b -m shell -a "systemctl is-active opensearch && systemctl is-enabled opensearch" --ask-vault-pass
```

---

### 3) Cluster health check (run from any node)

```bash
ansible -i inventory.ini os-a -b -m shell -a "curl -s http://127.0.0.1:9200/_cluster/health?pretty" --ask-vault-pass
```

Expected:

```json
"status": "green",
"number_of_nodes": 3
```

---

### 4) List nodes in the cluster

```bash
ansible -i inventory.ini os-a -b -m shell -a "curl -s http://127.0.0.1:9200/_cat/nodes?v" --ask-vault-pass
```

---

## High Availability (HA) Testing

### Test: Stop one node and verify cluster remains available

Stop `os-c`:

```bash
ansible -i inventory.ini os-c -b -m shell -a "systemctl stop opensearch" --ask-vault-pass
```

Check health from `os-a`:

```bash
ansible -i inventory.ini os-a -b -m shell -a "curl -s http://127.0.0.1:9200/_cluster/health?pretty" --ask-vault-pass
```

Expected:

* Cluster still accessible 
* Status may become **yellow** (replica unassigned) 

Bring node back:

```bash
ansible -i inventory.ini os-c -b -m shell -a "systemctl start opensearch" --ask-vault-pass
```

Re-check health:

```bash
ansible -i inventory.ini os-a -b -m shell -a "sleep 20 && curl -s http://127.0.0.1:9200/_cluster/health?pretty" --ask-vault-pass
```

Expected:

* Cluster returns to **green** 
* Node count returns to **3** 

---

## Dashboards Access

If dashboards enabled, access:

```
http://<Dashboards_Node_PublicIP>:5601
```

---

## Troubleshooting

### 1) `Permission denied (publickey)` SSH issue

 Fix inventory user mismatch (Ubuntu uses `ubuntu`, Amazon/RHEL uses `ec2-user`)

### 2) RPM GPG signature errors

Some OS versions block SHA1-based signatures. Workarounds:

* import GPG key (preferred)
* fallback to skip signature check for lab environments (not production)

### 3) Cluster not forming

Verify these in `/etc/opensearch/opensearch.yml`:

* `cluster.name`
* `node.name`
* `discovery.seed_hosts`
* `cluster.initial_cluster_manager_nodes`
* `network.publish_host`

---

## Security Best Practices

- Use Ansible Vault for passwords
- Enable TLS for HTTP and Transport layers
- Avoid exposing port 9200 publicly (restrict to internal VPC / VPN / bastion)
- Enable RBAC and least privilege roles
- Separate node roles in production:

* Dedicated cluster-manager nodes
* Dedicated data nodes

---

## Author / Maintainer

Maintained by DevOps team for automated deployment of OpenSearch in multi-OS environments.

