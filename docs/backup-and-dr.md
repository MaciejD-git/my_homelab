# Backup and Disaster Recovery

## Overview

This document describes how virtual machines, containers and endpoints are backed up in the homelab, and how I approach verification and restore testing.

The goal is to be able to quickly recover from failures of individual services, nodes or the main storage system.

## VM and container backups – Proxmox Backup Server

### Design

- All Proxmox VE nodes back up their VMs and containers to a central Proxmox Backup Server (PBS) instance.
- PBS runs as a VM on storage provided by TrueNAS, so backups are stored outside of the Proxmox cluster itself.
- Backups are deduplicated and compressed to optimize storage usage.

> **Suggested screenshots:**
> - PBS dashboard showing datastore usage and recent tasks.
> - Proxmox backup job configuration for one node (schedule + target PBS datastore).

### Scheduling and retention

- Regular backup jobs run from each Proxmox node to PBS on a defined schedule (e.g. nightly).
- Retention is configured to keep a rolling history of snapshots (daily/weekly/monthly) rather than “keep everything forever”.
- The aim is to balance storage usage with the ability to roll back to several recent points in time.

> **Suggested screenshot:**
> - PBS datastore view showing backup groups and retention settings.

### Verification

- PBS verify jobs are scheduled to automatically check the integrity of newly created backups. [web:48][web:51]
- Periodic re-verification is configured to re-check older backups and catch potential bit rot or storage issues. [web:48][web:51]
- Verification logs are monitored, and failed verifies are treated as incidents requiring investigation.

> **Suggested screenshot:**
> - PBS “Verify Jobs” tab with an example job and recent run history.

## Endpoint backups – UrBackup

### Design

- UrBackup runs as a central backup server for desktops and laptops in the homelab.
- Endpoints use the UrBackup client to perform file-level backups of important directories to NAS storage.
- This complements PBS by protecting user data that is not inside VMs/containers. [web:52][web:61]

> **Suggested screenshots:**
> - UrBackup web UI showing list of clients and their status.
> - Example backup history for one endpoint.

### Scheduling and retention

- Backup schedules are configured per client (e.g. regular incremental backups with less frequent full backups). [web:55]
- Retention policies ensure that multiple restore points are available without growing indefinitely.
- The focus is on being able to recover user data from at least the last few weeks/months, depending on the endpoint.

> **Suggested screenshot:**
> - UrBackup settings for one client (schedule + retention).

## Restore strategy and testing

### Restore scenarios

I treat the following scenarios as primary recovery targets:

- Restore of a single VM or container from PBS after accidental misconfiguration or failure.
- Restore of individual files or directories from UrBackup for endpoints.
- Restore of core services (e.g. reverse proxy, DNS, monitoring) on a fresh VM if the original host is lost.

### Testing restores

- Test restores from PBS are performed periodically by restoring selected VMs/CTs to a temporary location and verifying that they boot and services start correctly. [web:56][web:53]
- UrBackup restores are tested by recovering files to a test path and checking data integrity. [web:52][web:61]
- Observations and any issues found during restore tests are used to adjust backup settings or documentation.

> **Suggested screenshots:**
> - Proxmox restore dialog for a VM/CT (with a test destination).
> - UrBackup restore screen for a file/directory restore.

## Documentation and DR notes

- Backup configuration (PBS jobs, verify jobs, UrBackup settings) is documented so it can be recreated if needed.
- Critical secrets for accessing PBS, UrBackup and storage are stored in Vaultwarden and backed up as part of the overall strategy.
- Future work includes formalizing disaster recovery runbooks (step-by-step procedures) for restoring core services on new hardware.

