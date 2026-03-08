# Core Services

## Overview

This document focuses on the core platform services that run on top of the compute and network layers: monitoring, logging, security tooling, backup frontends and automation.

Network-facing components like DNS filtering (AdGuard Home) and reverse proxy (Nginx Proxy Manager) are described in more detail in `docs/network.md`.

## Observability & alerts

### Zabbix – infrastructure monitoring

Zabbix is used for basic infrastructure monitoring of the homelab.

From an operational perspective:

- Uses primarily built-in templates to monitor Proxmox nodes, storage and selected services.
- Provides a quick overview of host status (up/down, CPU, RAM, disk usage).
- Generates email alerts for key issues (e.g. node unavailable, filesystem nearly full).

> **Suggested screenshots:**
> - Zabbix dashboard or host list showing your Proxmox/TrueNAS nodes.
> - Example graph or problem list.

### Uptime monitoring – Uptime Kuma + Pushover

Uptime Kuma complements Zabbix by focusing on simple, fast availability checks.

- Monitors reachability of all homelab nodes and key web services (HTTP(S), ICMP, TCP).
- Uses Pushover integration to send push notifications to a mobile app on incidents.
- Helps quickly detect issues like VPN or reverse proxy being down even before deeper metrics show up.

> **Suggested screenshots:**
> - Uptime Kuma monitor list with statuses for Proxmox/TrueNAS/Graylog/etc.
> - Notification configuration showing Pushover as a notification channel.

## Logging & security analytics

### Central logging – Graylog

Graylog is the central log aggregation point.

- Collects logs from servers, containers and selected applications.
- Provides dashboards and search capabilities for troubleshooting and correlation of events.
- Acts as the base layer for security-related log analysis together with Wazuh.

> **Suggested screenshots:**
> - Graylog main dashboard with overview of incoming logs.
> - Example search or dashboard focused on one component (e.g. Proxmox or WireGuard).

### SIEM / XDR – Wazuh

Wazuh adds SIEM/XDR capabilities on top of raw logs and agent telemetry.

- Wazuh server is deployed in the homelab; agents are installed on selected hosts.
- Used to experiment with security monitoring: detection rules, file integrity monitoring, suspicious activity alerts.
- Configuration (rules, dashboards, alerting) is an ongoing project and part of the learning goals.

> **Suggested screenshots:**
> - Wazuh dashboard showing security events over time.
> - Agents view listing connected systems.

## Identity & secrets

### Vaultwarden – password management

Vaultwarden provides self-hosted password and secret management.

- Stores credentials for homelab services, infrastructure logins and personal accounts.
- Protected with strong master password and 2FA.
- Included in the backup strategy to ensure secrets can be restored in case of disaster.

> **Suggested screenshots:**
> - Vaultwarden login screen.
> - Vault overview with all sensitive details blurred/hidden.

## Backup frontends

(See `docs/backup-and-dr.md` for detailed backup strategy; below focuses on the service layer.)

### Proxmox Backup Server – backup UI

Proxmox Backup Server (PBS) provides the interface and logic for VM/CT backups.

- Shows backup jobs, schedules and retention for all Proxmox nodes.
- Allows quick verification and test restores of selected guests.
- Runs on storage provided by TrueNAS, keeping backup infrastructure separate from the main Proxmox nodes.

> **Suggested screenshots:**
> - PBS dashboard showing datastore usage and recent tasks.
> - Backup task list for one node.

### UrBackup – endpoint backups

UrBackup is used for file-level backups from desktops and laptops.

- Agents run on endpoints and periodically back up important directories to the NAS.
- Provides a web UI to see backup status, history and to restore individual files or full directory trees.
- Complements VM/CT backups by protecting user data outside the virtual infrastructure.

> **Suggested screenshots:**
> - UrBackup web UI with list of clients and backup status.
> - Example backup history for one endpoint.

## Automation & workflows

### n8n – workflow automation

n8n is the main workflow automation tool in the homelab.

- Used to experiment with integrating services (APIs, webhooks, notifications).
- Suitable for building small automations (e.g. reacting to monitoring/logging events, scheduled tasks).
- Planned to be integrated more tightly with monitoring and security tooling over time.

> **Suggested screenshots:**
> - n8n main UI.
> - Simple example workflow graph.

### Ansible – configuration management (planned)

Ansible is planned for configuration management and provisioning.

- Will be used to codify server and service configuration in playbooks and roles.
- Fits into the long-term goal of making the homelab reproducible and easier to rebuild.

> **Suggested screenshot (later):**
> - Example playbook or inventory once Ansible is in active use.
