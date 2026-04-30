# OpenSearch & OpenSearch Dashboards Ansible Role

## Overview

This Ansible role automates the deployment and management of:

* OpenSearch Cluster (Highly Available)
* OpenSearch Dashboards
* TLS via Envoy Proxy
* Authentication (Username/Password)
* Backup & Restore using MinIO (S3 compatible)
* Fire Drill (Disaster Recovery validation)
* Cluster Health Verification

The role is designed to be modular, scalable, and production-ready.

---

## Repository Structure

```
.
├── defaults/
│   └── main.yml
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   ├── access-control.yml
│   ├── backup.yml
│   ├── config-dashboard.yml
│   ├── config-node.yml
│   ├── install-dashboard.yml
│   ├── install-exporter.yml
│   ├── install-node.yml
│   ├── metrics.yml
│   ├── prereqs.yml
│   ├── restore.yml
│   ├── s3-keystore.yml
│   ├── security-config.yml
│   ├── setup-Debian.yml
│   ├── setup-RedHat.yml
│   ├── verify-cluster.yml
│   └── main.yml
├── templates/
│   ├── opensearch.yml.j2
│   ├── opensearch_dashboards.yml.j2
│   ├── jvm.options.j2
│   ├── internal_users.yml.j2
│   ├── roles_mapping.yml.j2
│   ├── opensearch.env.j2
│   ├── opensearch-exporter.env.j2
│   └── opensearch-exporter.service.j2
├── vars/
└── .yamllint
```

---

## Features

### 1. High Availability (HA)

* Multi-node OpenSearch cluster
* Dedicated master/data roles support
* Fault-tolerant architecture

### 2. OpenSearch Dashboards

* UI for monitoring and visualization
* Integrated with OpenSearch cluster

### 3. Security (Authentication)

* Username/Password based login
* Managed via internal_users.yml

### 4. TLS via Envoy Proxy

* Secure communication using TLS
* Envoy acts as reverse proxy

### 5. Backup (MinIO)

* S3-compatible backup using MinIO
* Snapshot-based backups

### 6. Restore

* Restore cluster from MinIO snapshots

### 7. Fire Drill

* Simulate failure and recovery
* Validate DR readiness

### 8. Cluster Health Check

* Verifies cluster status (green/yellow/red)
* Ensures deployment success

---

## Prerequisites

* Ansible >= 2.12
* Supported OS:

  * Ubuntu / Debian
  * RHEL / CentOS / Amazon Linux
* Java (auto-installed via role)
* MinIO (for backup)

---

## Variables

Defined in `defaults/main.yml`

| Variable         | Description                |
| ---------------- | -------------------------- |
| cluster_name     | Name of OpenSearch cluster |
| node_roles       | master/data roles          |
| opensearch_port  | Default 9200               |
| dashboards_port  | Default 5601               |
| opensearch_minio_endpoint   | MinIO URL                  |
| opensearch_minio_access_key | Access key                 |
| opensearch_minio_secret_key | Secret key                 |
| tls_enabled      | Enable TLS via Envoy       |

---

## How to Use

### 1. Inventory Example

```
[opensearch]
node1 ansible_host=10.0.0.1
node2 ansible_host=10.0.0.2
node3 ansible_host=10.0.0.3

[dashboards]
dash1 ansible_host=10.0.0.10
```

### 2. Playbook Example

```
- hosts: all
  become: yes
  roles:
    - opensearch-role
```

### 3. Run Playbook

```
ansible-playbook -i inventory.ini deploy.yml
```

---

## Backup & Restore

### Backup

* Triggered via `backup.yml`
* Stores snapshots in MinIO

### Restore

* Triggered via `restore.yml`
* Recovers cluster state

---

## Fire Drill (Disaster Recovery)

Steps:

1. Take backup
2. Simulate failure
3. Restore from snapshot
4. Verify cluster health

---

## Cluster Verification

* Uses `verify-cluster.yml`
* Checks:

  * Node availability
  * Cluster status
  * API response

---

## Security

* Managed via:

  * `security-config.yml`
  * `internal_users.yml.j2`
  * `roles_mapping.yml.j2`

---

## OS-specific Setup

* Debian-based: `setup-Debian.yml`
* RHEL-based: `setup-RedHat.yml`

---

## Metrics & Monitoring

* Exporter installation via `install-exporter.yml`
* Metrics collection enabled

---

## Handlers

Located in `handlers/main.yml`

* Restart OpenSearch
* Restart Dashboards

---

## Best Practices

* Use separate master and data nodes
* Enable TLS in production
* Schedule periodic backups
* Perform regular fire drills

---

## Troubleshooting

| Issue             | Solution                  |
| ----------------- | ------------------------- |
| Cluster not green | Check node connectivity   |
| Backup fails      | Verify MinIO credentials  |
| TLS issues        | Check Envoy configuration |
| Auth failure      | Validate users config     |

---

## Future Improvements

* Auto-scaling support
* Integration with Kubernetes
* Advanced monitoring dashboards

---

## Author

Sneha Joshi
