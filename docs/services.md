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

<img width="1895" height="906" alt="image" src="https://github.com/user-attachments/assets/fe5e7938-3dac-4cc1-8598-5e04dc4d8b4b" />

## Patch management

### PatchMon – update visibility

PatchMon is used to monitor patch and package update status across the Linux-based infrastructure in the homelab.

- Provides a centralized view of pending updates for containers, virtual machines and hypervisors.
- Helps identify which systems require regular package updates or urgent security patching.
- Adds operational visibility that complements uptime and infrastructure monitoring.

This makes it easier to plan maintenance windows and keep the environment consistent over time.

<img width="1896" height="851" alt="image" src="https://github.com/user-attachments/assets/e836c018-10f0-4ca5-8a09-8fb2da47805e" />

### Uptime monitoring – Uptime Kuma + Pushover

Uptime Kuma complements Zabbix by focusing on simple, fast availability checks.

- Monitors reachability of all homelab nodes and key web services (HTTP(S), ICMP, TCP).
- Uses Pushover integration to send push notifications to a mobile app on incidents.
- Helps quickly detect issues like VPN or reverse proxy being down even before deeper metrics show up.

<img width="1901" height="833" alt="image" src="https://github.com/user-attachments/assets/2c09bd7a-0a29-441c-8660-15d7e115ea69" />

## Logging & security analytics

### Central logging – Graylog

Graylog is the central log aggregation point.

- Collects logs mainly from Unifi gateway
- Provides dashboards and search capabilities for troubleshooting and correlation of events.
- Acts as the base layer for security-related log analysis together with Wazuh.

Example Dashboard
<img width="1858" height="865" alt="image" src="https://github.com/user-attachments/assets/406d4e1b-9601-480b-95c6-01861c99b1a8" />


### SIEM / XDR – Wazuh

Wazuh adds SIEM/XDR capabilities on top of raw logs and agent telemetry.

- Wazuh server is deployed in the homelab; agents are installed on selected hosts.
- Used to experiment with security monitoring: detection rules, file integrity monitoring, suspicious activity alerts.
- Configuration (rules, dashboards, alerting) is an ongoing project and part of the learning goals.

List of users
<img width="1900" height="669" alt="image" src="https://github.com/user-attachments/assets/e3bae9c4-732f-489f-8b7c-8112f55e264d" />

Example Dashboard
<img width="1914" height="918" alt="image" src="https://github.com/user-attachments/assets/fb10bcd4-3eff-4f3b-8ff5-856c7bb5d865" />


## Identity & secrets

### Vaultwarden – password management

Vaultwarden provides self-hosted password and secret management.

- Stores credentials for homelab services, infrastructure logins and personal accounts.
- Protected with strong master password and 2FA based on ubikey.
- Included in the backup strategy to ensure secrets can be restored in case of disaster.

<img width="1902" height="868" alt="image" src="https://github.com/user-attachments/assets/9da3837c-0fe3-403b-8115-4a5e7284506a" />

## Backup frontends

(See `docs/backup-and-dr.md` for detailed backup strategy; below focuses on the service layer.)

### Proxmox Backup Server – backup UI

Proxmox Backup Server (PBS) provides the interface and logic for VM/CT backups.

- Shows backup jobs, schedules and retention for all Proxmox nodes.
- Allows quick verification and test restores of selected guests.
- Runs on storage provided by TrueNAS, keeping backup infrastructure separate from the main Proxmox nodes.

<img width="1794" height="915" alt="image" src="https://github.com/user-attachments/assets/da88dcec-47e5-40ed-bb99-d1c9677fb0ff" />

### Veeam Backup and Replication – endpoint backups

Veeam is used for file-level backups from desktops and laptops running Windows.

- Agents run on endpoints and periodically back up important directories to the NAS.
- Provides a web UI to see backup status, rescan status, jobs status and many other things.
- Complements VM/CT backups by protecting user data outside the virtual infrastructure.

<img width="905" height="518" alt="image" src="https://github.com/user-attachments/assets/ab1dcc41-f270-481a-91ec-1a062e32c377" />

## Automation & workflows

### n8n – workflow automation (planned)

n8n is planned as the main workflow automation tool in the homelab.

- Currently deployed but not yet used for production workflows.
- Intended for experimenting with integrations between services (APIs, webhooks, notifications).
- Will be used to recreat usefull workflows that could be used in companys's production enviorment

### Ansible – configuration management (planned)

Ansible is planned for configuration management and provisioning.

- Will be used to codify server and service configuration in playbooks and roles.
- Fits into the long-term goal of making the homelab reproducible and easier to rebuild.
