# Homelab

Technical documentation of my personal homelab used as a learning, testing and hardening environment.

## Goals

- Practice designing, operating and troubleshooting a multi-node virtualization cluster.
- Apply modern network security concepts (segmentation, zero-trust mindset, VPN, logging, SIEM).
- Maintain reproducible infrastructure for experiments in automation, monitoring and incident response.

## High-level architecture

- **Compute:** 3-node Proxmox VE cluster running on low-power mini PCs, plus a dedicated TrueNAS Scale server acting as storage and services host.  
- **Network:** UniFi Cloud Gateway and managed switch, segmented into multiple VLANs and protected by a default-deny firewall strategy inspired by zero-trust principles 
- **Remote access:** Self-hosted WireGuard server running on UniFi Cloud Gateway, with dynamic DNS updates via Cloudflare to handle changing public IP addresses.  
- **Storage & backup:** Central NAS with ZFS and dedicated backup infrastructure (Proxmox Backup Server for VM/LXC and UrBackup for file-level backups from endpoints).  
- **Security & observability:** Centralized logging (Graylog), Wazuh SIEM (in progress), multi-factor authentication, strict access control to management interfaces.  
- **Monitoring & alerts:** Zabbix for infrastructure metrics, Uptime Kuma for availability checks of all nodes and key services, Pushover mobile notifications for incidents.

## Key components

### Compute & virtualization

- 3-node Proxmox VE cluster hosting:
  - Core infrastructure services (DNS filtering, reverse proxy, monitoring, logging, password manager).
  - Application containers (e.g. automation, dashboards, documentation). 
- TrueNAS Scale server:
  - Provides storage and runs selected apps/VMs (backup server, monitoring tools, etc.). 

### Networking & remote access

- UniFi-based network with VLANs separating:
  - Management, servers, admin devices, Wi‑Fi clients and test/isolated networks. 
- Zone-based firewall rules enforcing default-deny between segments and only explicitly allowing necessary flows (DNS, HTTPS via reverse proxy, VPN, management access). 
- WireGuard VPN:
  - Terminates on UniFi Cloud Gateway.
  - Dynamic DNS via Cloudflare keeps a stable hostname pointing to a changing public IP.  

### Security, logging and monitoring

- **Central logging:**  
  - Graylog aggregates logs from infrastructure and services for troubleshooting and security analysis.
- **SIEM / XDR:**  
  - Wazuh deployed and agents connected; configuration and use-cases (alert rules, dashboards) are being expanded as part of ongoing work. 
- **Monitoring:**  
  - Zabbix for host and service metrics using built-in templates.  
  - Uptime Kuma to continuously check reachability of all nodes and key services.
  - Pushover integration used by Uptime Kuma to push incident notifications to a mobile device.
- **Access control and hardening:**  
  - No inbound ports are exposed directly to the internet; all remote access goes through WireGuard.  
  - Management interfaces are restricted to specific VLANs and clients.
  - 2FA/YubiKey and least-privilege accounts for administrative access where supported. 

## Backup and recovery

- Proxmox Backup Server:
  - Deduplicated backups of all Proxmox VMs and containers on storage hosted by TrueNAS. 
  - Scheduled backups with retention and periodic verification jobs to ensure restore integrity.
- Endpoint backups:
  - File-level backups from workstations to NAS.

## Documentation

This repository focuses on architecture and operations rather than full configuration exports.

- `docs/network.md` – VLAN design, firewall strategy and remote access.  
- `docs/compute.md` – Proxmox cluster, TrueNAS and main workloads.  
- `docs/services.md` – Core services (DNS filtering, reverse proxy, monitoring, logging, automation).  
- `docs/backup-and-dr.md` – Backup schedules, retention and restore approach.  
- `docs/security.md` – Access control, MFA, logging, SIEM and zero-trust principles.

(Specific paths may evolve as the homelab grows.)

## Roadmap

Planned improvements and learning objectives:

- Enable **high availability** across all 3 Proxmox nodes for selected critical workloads. 
- Finalize **Wazuh** configuration (rules, dashboards, alerting workflows) to strengthen security monitoring and incident response capabilities. 
- Introduce more **automation** using n8n for workflows and Ansible for configuration management and provisioning. 

> Sensitive data (passwords, private keys, detailed configs) is intentionally omitted. The focus here is on design, operations and security practices.
