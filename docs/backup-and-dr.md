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
- `serwerproxmox3`   
  - 01:30 – backup of all VMs and containers (daily).

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

## Endpoint backups – Veeam Backup & Replication

### Design

- End-user backup for the Windows workstation is handled by Veeam Backup & Replication Community Edition.
- The Veeam server runs as a Windows Server 2025 VM on Proxmox and is kept in a workgroup, not joined to a domain.
- The backup repository is hosted on TrueNAS and exposed over SMB.
- The protected client is managed through a Veeam protection group and agent-based workflow.

### Access and deployment model

- The client uses a local administrator account with full rights.
- Remote access for deployment and management relies on hostname/IP reachability and the required Windows remote management services being available.
- Backup agent deployment is handled by Veeam, with Changed Block Tracking enabled where supported.
- Automatic updates for installed components are enabled to keep the agent and related components current.
- Automatic reboot after deployment is left disabled so restarts remain under manual control.

### Protection group and discovery

- The protected computer is included in a Veeam protection group.
- Discovery / rescan can be scheduled daily or at another low frequency, since this is a single workstation and does not require constant polling.
- The protection group is configured to install the backup agent and, where useful, the Changed Block Tracking driver.
- CDP agent deployment is disabled, since the workstation does not need replication or CDP features.
- After deployment, a reboot is performed if Veeam reports that it is required, and then the rescan is repeated until it completes cleanly.

### Backup policy

- Backups run as forward incremental jobs with synthetic full backups created periodically.
- Active full backups are disabled to avoid unnecessary load on the workstation and network.
- The job is scheduled once per day, tied to typical workstation usage.
- If the computer is powered off at the scheduled time, the backup runs when the machine is next powered on.
- After the backup finishes, the machine is left running.

### Retention and maintenance

- The retention policy keeps backups for 14 days, excluding days when no backup was taken.
- In practice, this gives roughly two weeks of restore points for a regularly used PC.
- Storage-level corruption guard / health check is enabled to verify backup integrity and catch damaged blocks.
- Full backup file maintenance such as defragmentation and compacting is disabled, because the benefit is small in this setup compared to the extra I/O and space usage.

### Restore strategy

- The main restore target is the Windows workstation itself and its user data.
- The solution is intended to provide quick recovery from accidental deletion, file corruption, workstation failure or other endpoint incidents.
- Restore tests should be documented once the first full workflow is validated.

<img width="903" height="507" alt="image" src="https://github.com/user-attachments/assets/1f564c53-3b54-4e4b-872b-1d773f262dde" />
