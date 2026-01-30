# OpenSearch Ansible Role – Production-Ready HA Deployment

## Overview

This Ansible role installs and configures an **OpenSearch High Availability cluster** in a fully automated and repeatable way.

The role supports:

* Multi-node cluster deployments
* Multi-OS support (RedHat / Amazon Linux / CentOS and Debian based systems)
* Secure and non-secure deployments (TLS on/off)
* OpenSearch Security plugin
* OpenSearch Dashboards
* SSO (SAML / OIDC / JWT)
* Snapshot repositories (S3, filesystem, Azure)
* Fully idempotent runs
* Production-ready system tuning and bootstrap settings

The role is designed to eliminate all manual steps and configuration drift.

---

## Table of Contents

* [Architecture & Design Goals](#architecture--design-goals)
* [Supported Platforms](#supported-platforms)
* [Features Checklist](#features-checklist)
* [Role Structure](#role-structure)
* [Inventory & Group Variables](#inventory--group-variables)
* [Deployment Modes](#deployment-modes)

  * [Without TLS](#without-tls)
  * [With TLS](#with-tls)
* [Security Plugin](#security-plugin)
* [Dashboards](#dashboards)
* [SSO Configuration](#sso-configuration)
* [Snapshot Repository](#snapshot-repository)
* [High Availability & Cluster Formation](#high-availability--cluster-formation)
* [Variables Reference (Important Ones)](#variables-reference-important-ones)
* [Execution](#execution)
* [Post-Deployment Validation](#post-deployment-validation)
* [Operational Notes](#operational-notes)
* [Known Limitations / Notes](#known-limitations--notes)

---

## Architecture & Design Goals

This role is built for:

* Infrastructure-as-Code
* Repeatable deployments
* No manual configuration on servers
* Support for both secure and non-secure clusters
* Easy enable / disable of features through group variables

---

## Supported Platforms

* Amazon Linux 2 / Amazon Linux 2023
* RHEL / CentOS / Rocky / Alma
* Debian / Ubuntu

Package installation is handled automatically using the correct repository for each platform.

---

## Features Checklist

The following checklist represents what is implemented and validated in this role.

### Core

* [x] OpenSearch installation
* [x] OpenSearch Dashboards installation
* [x] Version pinning
* [x] Multi-node cluster support
* [x] Automatic discovery and cluster bootstrapping
* [x] Idempotent runs

### System & OS Tuning

* [x] vm.max_map_count
* [x] file descriptor limits
* [x] nproc limits
* [x] memlock limits
* [x] systemd override configuration
* [x] JVM tuning
* [x] log4j configuration

### Security

* [x] OpenSearch Security plugin support
* [x] Internal users
* [x] Roles mapping
* [x] Optional anonymous authentication
* [x] Admin user bootstrap
* [x] Supports TLS and non-TLS modes

### TLS / SSL

* [x] HTTP TLS (client to cluster)
* [x] Transport TLS (node-to-node)
* [x] Certificate deployment via Ansible
* [x] Non-TLS mode supported

### Dashboards

* [x] Dashboards installation
* [x] Dashboards configuration
* [x] Optional Dashboards TLS
* [x] Dashboards service user integration

### SSO

* [x] SAML
* [x] OIDC
* [x] JWT
* [x] Role mapping for SSO users
* [x] Dashboards SSO configuration

### Snapshot Repositories

* [x] S3 repository
* [x] Filesystem repository
* [x] Azure repository
* [x] Keystore handling

---

## Role Structure

```
opensearch/
├── defaults/
│   └── main.yml
├── handlers/
│   └── main.yml
├── tasks/
│   ├── prerequisites.yml
│   ├── install_opensearch.yml
│   ├── configure_opensearch.yml
│   ├── security_setup.yml
│   ├── install_dashboards.yml
│   ├── configure_dashboards.yml
│   ├── sso_setup.yml
│   ├── snapshot_setup.yml
│   └── main.yml
├── templates/
│   ├── opensearch.yml.j2
│   ├── config.yml.j2
│   ├── internal_users.yml.j2
│   ├── roles_mapping.yml.j2
│   ├── jvm.options.j2
│   └── opensearch-dashboards.yml.j2
└── vars/
    ├── Debian.yml
    ├── RedHat.yml
    └── main.yml
```

---

## Inventory & Group Variables

This role is designed to be driven mainly from:

```
group_vars/all.yml
```

and inventory:

```
inventory.ini
```

Example inventory:

```ini
[opensearch]
os-a
os-b
os-c
```

---

## Deployment Modes

This role supports two main deployment models.

---

### Without TLS

This is useful for:

* internal trusted networks
* labs
* testing environments

In `group_vars/all.yml`:

```yaml
opensearch_security_enabled: false

opensearch_enable_tls: false
opensearch_http_ssl_enabled: false
opensearch_transport_ssl_enabled: false
```

---

### With TLS

In `group_vars/all.yml`:

```yaml
opensearch_security_enabled: true

opensearch_enable_tls: true
opensearch_http_ssl_enabled: true
opensearch_transport_ssl_enabled: true
```

Certificates must be supplied using vault variables:

```yaml
opensearch_ssl_cert: ""
opensearch_ssl_key: ""
opensearch_ssl_ca: ""

opensearch_admin_cert: ""
opensearch_admin_key: ""
```

---

## Security Plugin

The role deploys the OpenSearch security plugin configuration using:

* `config.yml`
* `internal_users.yml`
* `roles_mapping.yml`

The following items are intentionally not managed by default:

* roles.yml
* action_groups.yml
* tenants.yml

This avoids accidental privilege or tenant changes in production.

---

## Dashboards

Dashboards are enabled using:

```yaml
opensearch_dashboards_enabled: true
```

Dashboards connect to OpenSearch using the service account:

```yaml
opensearch_kibanaserver_username
opensearch_kibanaserver_password
```

TLS for Dashboards is controlled independently:

```yaml
opensearch_dashboards_enable_tls
```

---

## SSO Configuration

SSO is controlled through:

```yaml
opensearch_sso_enabled: true
opensearch_sso_type: saml   # saml | oidc | jwt
```

Supported providers:

* SAML
* OpenID Connect
* JWT

The role automatically:

* injects SSO configuration into `config.yml`
* updates role mappings
* configures Dashboards for the selected provider

---

## Snapshot Repository

Snapshots can be enabled using:

```yaml
opensearch_snapshot_enabled: true
opensearch_snapshot_type: s3    # s3 | fs | azure
```

The role automatically:

* installs the required repository plugin
* configures keystore credentials
* registers the snapshot repository

---

## High Availability & Cluster Formation

The role automatically:

* builds the discovery seed hosts
* configures cluster manager nodes
* handles `cluster.initial_cluster_manager_nodes`

The cluster manager list is derived from:

```yaml
groups['opensearch']
```

This ensures that new nodes are automatically included without manual reconfiguration.

---

## Variables Reference (Important Ones)

### Core

```yaml
opensearch_cluster_name
opensearch_node_name
opensearch_network_host
opensearch_network_publish_host
```

---

### Security & TLS

```yaml
opensearch_security_enabled

opensearch_enable_tls
opensearch_http_ssl_enabled
opensearch_transport_ssl_enabled
```

---

### Dashboards

```yaml
opensearch_dashboards_enabled
opensearch_dashboards_enable_tls
```

---

### SSO

```yaml
opensearch_sso_enabled
opensearch_sso_type
```

---

### Snapshots

```yaml
opensearch_snapshot_enabled
opensearch_snapshot_type
```

---

## Execution

From the project root:

```bash
ansible-playbook site.yml -i inventory.ini --ask-vault-pass
```

---

## Post-Deployment Validation

Check cluster health:

```bash
curl http://<node-ip>:9200/_cluster/health?pretty
```

If security is enabled:

```bash
curl -u admin:<password> http://<node-ip>:9200/_cluster/health?pretty
```

Dashboards:

```
http://<node-ip>:5601
```

---

## Operational Notes

* The role restarts OpenSearch only when configuration changes.
* systemd overrides are used to avoid modifying vendor unit files.
* TLS and non-TLS paths are fully separated using conditions.
* SSO configuration is additive and managed using markers.
* All sensitive variables should be stored using Ansible Vault.

---

## Known Limitations / Notes

* Certificate generation is not handled by this role.
* This role assumes network connectivity between all cluster nodes.
* Snapshot repository creation requires the cluster to be healthy.
* SSO providers must be prepared externally (IdP / OAuth provider).

---

## Recommended Usage Pattern

Use:

```
group_vars/all.yml
```

as the single control point for:

* enabling TLS
* enabling SSO
* enabling snapshots
* enabling Dashboards

This keeps the role reusable across environments:

* dev
* staging
* production
