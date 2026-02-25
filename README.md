# OpenSearch Ansible Role

---

## What is OpenSearch?

OpenSearch is an open-source, community-driven search and analytics engine forked from Elasticsearch 7.10. It provides:

* Full-text search and log analytics
* Real-time indexing and querying
* Built-in security plugin with TLS, RBAC, and audit logging
* OpenSearch Dashboards for visualization and exploration
* Plugin-based architecture (alerting, anomaly detection, SQL, etc.)

Official docs: [https://opensearch.org/docs/latest/](https://opensearch.org/docs/latest/)

---

## Purpose of This Role

The objective of this role is to:

* Install OpenSearch and OpenSearch Dashboards consistently across multiple OS families
* Support both single-node and multi-node (HA) cluster deployments
* Provide two security modes — **TLS** (production) and **HTTP with demo certs** (development)
* Enable optional SSO integration (OpenID Connect / SAML)
* Configure custom RBAC (roles, role mappings, internal users) via the Security API
* Optionally enable Prometheus metrics and Performance Analyzer
* Follow Ansible best practices (FQCN, handlers, OS detection, idempotency)

---

## Supported Operating Systems

| OS Family    | Versions                    |
| ------------ | --------------------------- |
| Debian       | Bullseye, Bookworm          |
| Ubuntu       | Jammy (22.04), Noble (24.04)|
| RedHat / EL  | 8, 9                        |
| Amazon Linux | All                         |

OS detection is done using `ansible_os_family` and `ansible_distribution`.

---

## Prerequisites

### System Requirements

| Requirement      | Description                                    |
| ---------------- | ---------------------------------------------- |
| RAM              | 4 GB minimum (8 GB+ recommended for production)|
| CPU              | 2 vCPUs or more                                |
| Disk             | SSD-backed storage recommended                 |
| Access           | SSH + sudo                                     |
| Internet         | Required (to download packages from repos)     |
| vm.max_map_count | ≥ 262144 (configured automatically by the role)|

### Python / Ansible Requirements

| Package               | Purpose                |
| --------------------- | ---------------------- |
| ansible >= 2.14        | Playbook execution     |
| ansible.posix collection | sysctl module        |
| boto3 & botocore      | AWS dynamic inventory (if applicable) |

Install:

```bash
pip install ansible boto3 botocore
ansible-galaxy collection install ansible.posix
```

---

## Virtual Environment Setup

```bash
python3 -m venv ansible-venv
source ansible-venv/bin/activate
pip install ansible boto3 botocore
ansible-galaxy collection install ansible.posix
```

---

## Role Directory Structure

```
opensearch/
├── defaults/main.yml          # All user-configurable variables
├── vars/
│   ├── main.yml               # Internal derived variables
│   ├── Debian.yml             # Debian/Ubuntu-specific packages
│   ├── RedHat.yml             # RHEL/CentOS-specific packages
│   └── Amazon.yml             # Amazon Linux-specific packages
├── tasks/
│   ├── main.yml               # Orchestration entrypoint
│   ├── prereqs.yml            # Java, kernel tuning, swap
│   ├── setup-Debian.yml       # APT dependency installation
│   ├── setup-RedHat.yml       # YUM dependency installation
│   ├── install-node.yml       # OpenSearch package installation
│   ├── config-node.yml        # Config files, systemd, security plugin
│   ├── security-config.yml    # TLS mode: certs, securityadmin.sh
│   ├── security-config-http.yml # HTTP mode: verify auth
│   ├── access-control.yml     # Custom roles, mappings, users via API
│   ├── metrics.yml            # Prometheus / Performance Analyzer
│   ├── install-dashboard.yml  # Dashboards package installation
│   ├── config-dashboard.yml   # Dashboards configuration + service
│   └── verify-cluster.yml     # Health check + default index template
├── templates/
│   ├── opensearch.yml.j2
│   ├── opensearch_dashboards.yml.j2
│   ├── jvm.options.j2
│   ├── opensearch.env.j2
│   ├── opensearch_override.conf.j2
│   ├── internal_users.yml.j2
│   ├── roles_mapping.yml.j2
│   └── default_template.json.j2
├── handlers/main.yml
└── meta/main.yml
```

---

## Key Variables

### Required (Must Be Set)

| Variable                  | Description                                  |
| ------------------------- | -------------------------------------------- |
| `opensearch_domain_name`  | Domain or IP used for API calls and health checks. **No default — role will fail if unset.** |

### Version & Installation Toggles

| Variable                         | Default              | Description                              |
| -------------------------------- | -------------------- | ---------------------------------------- |
| `opensearch_version`             | `"2.11.1"`           | OpenSearch version to install            |
| `opensearch_install_node`        | `true`               | Install the OpenSearch engine            |
| `opensearch_install_dashboard`   | `true`               | Install OpenSearch Dashboards            |
| `opensearch_enable_cluster_mode` | `true`               | Enable multi-node cluster discovery      |

### Cluster

| Variable                             | Default                                   | Description                        |
| ------------------------------------ | ----------------------------------------- | ---------------------------------- |
| `opensearch_cluster_name`            | `"opensearch-cluster"`                    | Cluster name                       |
| `opensearch_node_roles`              | `[cluster_manager, data, ingest]`         | Roles assigned to this node        |
| `opensearch_discovery_seed_hosts`    | `groups['opensearch']`                    | Auto-discovered from inventory     |
| `opensearch_initial_master_nodes`    | `groups['opensearch']`                    | Auto-discovered from inventory     |

### Network

| Variable                    | Default     | Description               |
| --------------------------- | ----------- | ------------------------- |
| `opensearch_network_host`   | `"0.0.0.0"` | Listen address           |
| `opensearch_http_port`      | `9200`      | REST API port             |
| `opensearch_transport_port` | `9300`      | Node-to-node transport    |

### Security / TLS

| Variable                | Default        | Description                                         |
| ----------------------- | -------------- | --------------------------------------------------- |
| `opensearch_use_tls`    | `false`        | `true` = production TLS, `false` = HTTP + demo certs|
| `opensearch_admin_username` | `"admin"`  | Admin username                                      |
| `opensearch_admin_password` | `"Strong@321"` | **Change this!** Use Ansible Vault in production |

### Dashboards

| Variable                       | Default     | Description                   |
| ------------------------------ | ----------- | ----------------------------- |
| `opensearch_dashboards_port`   | `5601`      | Dashboards listen port        |
| `opensearch_dashboards_host`   | `"0.0.0.0"` | Dashboards bind address      |

See `defaults/main.yml` for the full list of configurable variables including JVM heap, system tuning, SSO, access control, and metrics options.

---

## Inventory Setup

The role auto-discovers cluster peers from the `opensearch` inventory group. Example inventory:

```ini
[opensearch]
node1.example.com
node2.example.com
node3.example.com
```

For a single-node deployment, define one host or set `opensearch_enable_cluster_mode: false`.

---

## Usage Examples

### Minimal Single-Node (HTTP Mode)

```yaml
- hosts: opensearch
  become: true
  roles:
    - role: opensearch
      vars:
        opensearch_domain_name: "{{ ansible_default_ipv4.address }}"
        opensearch_enable_cluster_mode: false
```

### Production Multi-Node Cluster (TLS Mode)

```yaml
- hosts: opensearch
  become: true
  roles:
    - role: opensearch
      vars:
        opensearch_domain_name: "opensearch.example.com"
        opensearch_use_tls: true
        opensearch_admin_password: "{{ vault_opensearch_admin_password }}"
        opensearch_kibanaserver_password: "{{ vault_opensearch_kibana_password }}"
        opensearch_root_ca_path: "/etc/opensearch/certs/chain.pem"
        opensearch_node_cert_path: "/etc/opensearch/certs/fullchain.pem"
        opensearch_node_key_path: "/etc/opensearch/certs/privkey.pem"
        opensearch_admin_dn:
          - "CN=admin,OU=MyOrg,O=MyOrg,L=City,ST=State,C=US"
        opensearch_nodes_dn:
          - "CN=node,OU=MyOrg,O=MyOrg,L=City,ST=State,C=US"
        opensearch_jvm_heap_size: "4g"
```

### With Custom RBAC and Metrics

```yaml
- hosts: opensearch
  become: true
  roles:
    - role: opensearch
      vars:
        opensearch_domain_name: "opensearch.example.com"
        opensearch_access_control_enabled: true
        opensearch_custom_roles:
          - name: "readall_role"
            cluster_permissions: ["cluster_composite_ops_ro"]
            index_permissions:
              - index_patterns: ["*"]
                allowed_actions: ["read"]
        opensearch_custom_internal_users:
          - name: "readonly_user"
            password_hash: "$2a$12$..."
            backend_roles: ["readall"]
        opensearch_metrics_enabled: true
        opensearch_performance_analyzer_enabled: true
```

---

## Running the Playbook

```bash
ansible-playbook -i inventory/hosts opensearch.yml
```

Expected:

```
failed=0
```

---

## Verification

### Service Status

```bash
sudo systemctl status opensearch
sudo systemctl status opensearch-dashboards
```

### Cluster Health (HTTP mode)

```bash
curl -u admin:Strong@321 http://localhost:9200/_cluster/health?pretty
```

### Cluster Health (TLS mode)

```bash
curl -u admin:<password> --cacert /etc/opensearch/chain.pem https://localhost:9200/_cluster/health?pretty
```

### Dashboards

```
http://<SERVER_IP>:5601
```

---

## Best Practices Implemented

| Practice                    | Description                                          |
| --------------------------- | ---------------------------------------------------- |
| FQCN                        | Uses `ansible.builtin.*` for all modules             |
| Handlers                    | Clean systemd reload and service restart             |
| Flush handlers              | Ensures OpenSearch is running before API calls        |
| OS Detection                | Installs OS-specific packages per family             |
| Idempotency                 | Marker file prevents redundant security init          |
| Security                    | `no_log` on password-handling tasks                  |
| Retry Logic                 | Retries on package installs and API health checks    |
| Feature Toggles             | TLS, Dashboards, RBAC, and metrics are independently togglable |

---

## Troubleshooting

| Issue                           | Fix                                                           |
| ------------------------------- | ------------------------------------------------------------- |
| OpenSearch not starting         | `journalctl -u opensearch -n 100`                             |
| Cluster stuck on yellow/red     | Check `/_cluster/health?pretty` and node connectivity          |
| TLS certificate errors          | Verify cert paths and permissions (`0644` certs, `0600` keys) |
| `vm.max_map_count` too low      | `sysctl -w vm.max_map_count=262144` (role does this automatically) |
| Dashboards can't connect        | Verify `opensearch_dashboards_opensearch_host` and credentials |
| apt/dpkg lock                   | Role has built-in apt lock wait; or run `sudo dpkg --configure -a` |
| Security init needs re-run      | Delete `/etc/opensearch/.security_initialized` marker file     |

---

## Conclusion

This role provides a fully automated, production-ready setup of OpenSearch and OpenSearch Dashboards across Debian and RedHat-based systems. It supports single-node and multi-node clusters, TLS and HTTP security modes, SSO integration, custom RBAC, and Prometheus metrics — all controlled through simple variables.

---

## Reference Links

| Purpose                         | Link                                                                      |
| ------------------------------- | ------------------------------------------------------------------------- |
| OpenSearch Official Docs        | [https://opensearch.org/docs/latest/](https://opensearch.org/docs/latest/) |
| Security Plugin Guide           | [https://opensearch.org/docs/latest/security/](https://opensearch.org/docs/latest/security/) |
| Ansible Official Docs           | [https://docs.ansible.com](https://docs.ansible.com)                      |

---

**Author:** Sneha Joshi

**Company:** Opstree

**Last Updated on:** 24-Feb-2026

---
