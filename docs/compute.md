# Compute & Virtualization

## Overview

This document summarizes the compute layer of the homelab: the Proxmox VE cluster and the TrueNAS Scale server that together host the core services and workloads.  

The focus is on how the nodes are organized, what they run, and how they are monitored and backed up.

## Proxmox VE cluster

The homelab uses a 3-node Proxmox VE cluster built on 2 low-power mini pc and 1 hp z240 sff

Key points:

- All three nodes participate in the same Proxmox cluster.
- The cluster hosts core infrastructure services (DNS filtering, reverse proxy, monitoring, logging, backup, password manager, etc.).
- Workloads are a mix of lightweight LXC containers and full VMs where needed.

<img width="314" height="696" alt="image" src="https://github.com/user-attachments/assets/92dce489-55bf-49c2-a9fb-7eaf22b6911d" />

### Example workloads

Some of the main workloads running on the cluster:

- DNS filtering (AdGuard Home, redundant instances).  
- Reverse proxy and SSL termination (Nginx Proxy Manager).  
- Monitoring (Zabbix server).  
- Central logging (Graylog).  
- Password manager (Vaultwarden). 
- Automation and orchestration (n8n, Ansible control node – in progress).  

<img width="1341" height="891" alt="image" src="https://github.com/user-attachments/assets/e50b3c9b-00db-450f-89d6-5ec56955f281" />

## TrueNAS Scale

A dedicated TrueNAS Scale server complements the Proxmox cluster:

- Provides network storage for backups and selected workloads.  
- Hosts applications and VMs such as:
  - Proxmox Backup Server.  
  - File-level backup solution for endpoints.  
  - Disk health and SMART monitoring.  

<img width="1100" height="807" alt="image" src="https://github.com/user-attachments/assets/182e9de3-c4d8-4ca9-a9fc-d6e49438ae6b" />

## Backup integration

Proxmox and TrueNAS are tightly integrated for backup purposes:

- Proxmox Backup Server runs as a VM on TrueNAS storage and receives deduplicated backups from all three Proxmox nodes.  
- TrueNAS exposes storage for backup datastores and additional file-level backups.  

<img width="1905" height="596" alt="image" src="https://github.com/user-attachments/assets/c7e7f32c-9d5c-425b-8ea4-1a30ed4008b2" />

---

This page intentionally stays high-level. More detailed workload descriptions are provided in `docs/services.md` and `docs/backup-and-dr.md`.
