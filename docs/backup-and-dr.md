# Backup and Disaster Recovery

## Overview

TThis document describes how virtual machines, containers and endpoints are backed up in the homelab, and how backup integrity is verified over time.

The main goals are:

- Be able to quickly recover from failures of individual services, nodes or the main storage.
- Detect backup/storage issues early through regular verification and SMART/ZFS checks.
  
## VM and container backups – Proxmox Backup Server

### Design

- All Proxmox VE nodes back up their VMs and containers to a central Proxmox Backup Server (PBS) instance.
- PBS runs as a VM on storage provided by TrueNAS, so backups are stored outside of the Proxmox cluster itself.
- Backups are deduplicated and compressed to optimize storage usage.

<img width="1885" height="490" alt="image" src="https://github.com/user-attachments/assets/5eb64b80-ed98-4867-af06-7671eb37bb6c" />

### Backup schedule

Daily backup jobs run from each Proxmox node to PBS:

- `serwerproxmox`  
  - 01:00 – backup of all VMs and containers (daily).  
- `serwerproxmox2`  
  - 01:15 – backup of all VMs and containers (daily).  
  - 01:30 – backup of all VMs and containers (daily, additional window for specific workloads if needed).

This staggered schedule reduces load on PBS and storage by not starting all jobs at the same time.

<img width="1423" height="145" alt="image" src="https://github.com/user-attachments/assets/b9df8ccf-ef96-407f-a4ea-7cea4e8a43b7" />

### Retention

PBS retention is configured to keep a rolling history of snapshots (daily/weekly/monthly) instead of storing every backup indefinitely.

- Recent backups (last days) are kept densely.
- Older backups are pruned according to the retention policy to free space while preserving meaningful restore points.

<img width="724" height="274" alt="image" src="https://github.com/user-attachments/assets/cdd49f09-2e52-4bbd-8ad9-549f936d3f21" />

## Backup maintenance and verification

To ensure backups remain valid and storage healthy, additional maintenance jobs are scheduled.

### Garbage collection on PBS

- Every day at 02:00 – PBS runs a **garbage collection** job.
- This physically removes data from pruned snapshots and reclaims storage space.

### Verify jobs (new backups)

- Every day at 03:00 – PBS runs a **verify job** for backups created between 01:00 and 01:40.
- The verify job checks the integrity of newly created backup chunks and ensures they can be read back correctly.
- Failed verifies are treated as incidents and require investigation (storage issues, network problems, etc.).

### Periodic re-verify

- Every 14 days at 03:00 – a **re-verify** job is executed, re-verifying all existing backups on PBS. 
- This helps detect long-term issues such as bit rot or silent storage corruption.

in the right corner you see a message that verification has been successfully completed
<img width="1700" height="409" alt="image" src="https://github.com/user-attachments/assets/bc981623-6fc4-4768-8b18-676d6ba7e54e" />

## Storage health – TrueNAS (SMART & ZFS)

TrueNAS is responsible for the underlying storage that hosts PBS data and other services. To keep it healthy, regular SMART and ZFS jobs are scheduled.

### Weekly checks

- Every Sunday at 00:00 – a **ZFS scrub** is run on the TrueNAS pool.  
  - Scrub checks data integrity and fixes correctable errors on disk.  
- Every Sunday at 04:00 – a cron job triggers a **SMART short test** on all disks.  
  - This ensures that when Scrutiny reads SMART data, it always works with fresh test results.

### Monthly checks

- On the 1st day of each month at 05:00 – a cron job runs a **SMART long test** on all disks.  
  - This is a more thorough surface scan and can take several hours, but helps detect early disk issues.

<img width="1395" height="884" alt="image" src="https://github.com/user-attachments/assets/cdfb6745-e0ce-4f09-a86a-b82bf408300b" />

## Endpoint backups – UrBackup

While Proxmox Backup Server protects virtual machines and containers, UrBackup is responsible for user data on physical endpoints.

- Covers Windows desktops and laptops, focusing on user profiles and important data directories rather than full system images.
- Acts as a second layer of protection in case data is lost or corrupted on a workstation (accidental deletion, ransomware, disk failure).
- Restores are typically done at the file/folder level, which is faster and more practical than rebuilding whole machines from images.

UrBackup’s detailed UI and configuration are described in `docs/services.md`. Here it is considered as part of the overall backup and recovery strategy.

### Scheduling and retention

- Backup schedules are configured per client (e.g. regular incremental backups with less frequent full backups).
- Retention policies ensure that multiple restore points are available without growing indefinitely.
- The focus is on being able to recover user data from at least the last few weeks/months, depending on the endpoint.

<img width="1172" height="811" alt="image" src="https://github.com/user-attachments/assets/56a39468-9c55-4760-831d-bef01daa9dd5" />
<img width="1174" height="647" alt="image" src="https://github.com/user-attachments/assets/46abb46f-5f55-4b63-a62a-ec81623fdb27" />

## Restore strategy and testing

Restore testing approach:

- For PBS:
  - A few times per year I pick a non-critical VM or container (for example a lab service) and restore it from PBS to a temporary VM/CT ID.
  - I then boot it, log in and verify that the main service(s) start correctly and that data is accessible.
- For UrBackup:
  - I periodically restore a sample of files from one endpoint to a test directory (not overwriting the originals) and compare them to the live copies.
  - This validates that backups are actually usable and not silently corrupted.
- If any issues are found during these tests (slow restores, missing data, verify errors), I review the backup configuration, schedules or retention and update the documentation accordingly.

## Documentation and DR notes

- Backup configuration (PBS jobs, verify jobs, UrBackup settings) is documented so it can be recreated if needed.
- Critical secrets for accessing PBS, UrBackup and storage are stored in Vaultwarden and backed up as part of the overall strategy.
- Future work includes formalizing disaster recovery runbooks (step-by-step procedures) for restoring core services on new hardware.

