# Ansible Linux Operations

Enterprise-style Ansible automation for Linux platform operations.

This repository owns day-2 Linux administration workflows: OS updates, DNS client configuration, Filebeat log forwarding, Logstash pipeline setup, Kibana setup, and HashiCorp Vault installation. It was refactored from lab playbooks into role-based automation with clear ownership boundaries, example inventory, dependency files, lint configuration, and CI.

## Scope

Owned here:

- Rocky/RHEL package updates and repository enablement
- DNS client configuration through NetworkManager
- Filebeat installation and Logstash forwarding
- Logstash syslog pipeline configuration
- Kibana installation and firewall access
- Vault installation and systemd service management

Owned by other repos:

- RHEL/Rocky 9 CIS hardening: `ansible-rhel9-cis-hardening`
- Prometheus, Grafana, node_exporter: `ansible-observability-stack`

## Repository Layout

```text
ansible-linux-operations/
|-- README.md
|-- ansible.cfg
|-- requirements.yml
|-- collections/
|   `-- requirements.yml
|-- inventories/
|   `-- example/
|       |-- hosts.yml
|       `-- group_vars/
|           `-- all/
|               `-- main.yml
|-- playbooks/
|   |-- site.yml
|   |-- update.yml
|   |-- dns.yml
|   |-- filebeat.yml
|   |-- logstash.yml
|   |-- kibana.yml
|   `-- vault.yml
|-- roles/
|   |-- linux_update/
|   |-- dns_client/
|   |-- filebeat_agent/
|   |-- logstash_server/
|   |-- kibana_server/
|   `-- vault_server/
|-- .config/
|   `-- ansible-lint.yml
`-- .github/
    `-- workflows/
        `-- ansible-ci.yml
```

## What This Replaced

This repo preserves the useful work from the old `linux_admin` repo while removing the basic lab layout:

- `update_linux.yml` became the `linux_update` role.
- `dns_update.yml` became the `dns_client` role.
- `filebeat_setup.yml` became the `filebeat_agent` role.
- `logstash_setup.yml` became the `logstash_server` role.
- `install_kibana.yml` became the `kibana_server` role.
- `vault_installation.yml` became the `vault_server` role.

The old repo mixed Linux operations, hardening, monitoring, secrets, and lab scripts. This repo narrows the operational scope and keeps monitoring/hardening in their own repositories.

## Prerequisites

- Ansible Core
- SSH access to target hosts
- Sudo access for privileged tasks
- Red Hat or Rocky Linux targets for most roles
- Valid package repositories for Elastic and HashiCorp software

Install dependencies:

```bash
ansible-galaxy install -r requirements.yml
```

## Running

Run from the repository root:

```bash
cd ansible-linux-operations
```

Validate inventory:

```bash
ansible-inventory -i inventories/example/hosts.yml --list
```

Run all default operations:

```bash
ansible-playbook -i inventories/example/hosts.yml playbooks/site.yml
```

Run individual workflows:

```bash
ansible-playbook -i inventories/example/hosts.yml playbooks/update.yml
ansible-playbook -i inventories/example/hosts.yml playbooks/dns.yml
ansible-playbook -i inventories/example/hosts.yml playbooks/filebeat.yml
ansible-playbook -i inventories/example/hosts.yml playbooks/logstash.yml
ansible-playbook -i inventories/example/hosts.yml playbooks/kibana.yml
ansible-playbook -i inventories/example/hosts.yml playbooks/vault.yml
```

Use `--limit` for controlled operations:

```bash
ansible-playbook -i inventories/example/hosts.yml playbooks/update.yml --limit postgresdb.example.com
```

## Configuration

Example variables live in:

```text
inventories/example/group_vars/all/main.yml
```

Important variables:

- `linux_update_enable_epel`
- `linux_update_enable_crb`
- `linux_update_reboot_if_required`
- `dns_client_interface`
- `dns_client_server`
- `filebeat_logstash_host`
- `filebeat_logstash_port`
- `logstash_elasticsearch_host`
- `kibana_elasticsearch_host`
- `vault_tls_disable`

Production values should be moved into environment-specific inventories and encrypted vault files where sensitive.

## Validation

Syntax checks:

```bash
for playbook in playbooks/*.yml; do
  ansible-playbook -i inventories/example/hosts.yml "$playbook" --syntax-check
done
```

Lint:

```bash
ansible-lint -c .config/ansible-lint.yml .
```

CI:

GitHub Actions runs dependency installation, `ansible-lint`, and syntax checks on pull requests and pushes to `main`.

## Enterprise Design Choices

- Role-based structure instead of one-off task files
- Example inventory separated from implementation
- Real dependency files for collections
- `.gitignore` blocks vault password files, secrets, generated collections, logs, and local environments
- CI workflow included
- Host key checking enabled by default
- Privilege escalation is play-scoped, not globally forced
- Vault defaults do not disable TLS unless explicitly configured

## Security Notes

Do not commit:

- Vault passwords
- Plaintext secrets
- SSH private keys
- TLS private keys
- Service account tokens

The original lab repo contained local secret-oriented files. This public version intentionally excludes secret material and documents where secret-backed variables should be supplied.

## Troubleshooting

### Package Install Fails

Check repository reachability, subscription status, proxy configuration, and OS family.

### Filebeat Cannot Reach Logstash

Check `filebeat_logstash_host`, `filebeat_logstash_port`, firewalls, and Logstash pipeline status.

### Kibana Fails to Start

Check the rendered `/etc/kibana/kibana.yml`, Elasticsearch connectivity, and port `5601/tcp`.

### Vault Is Unhealthy

Check TLS variables, listener address, storage path permissions, systemd logs, and initialization/unseal state.

## Portfolio Notes

This repo demonstrates Linux operations ownership, refactoring lab playbooks into roles, clean dependency management, and practical production concerns without pretending that example inventory is production inventory.
