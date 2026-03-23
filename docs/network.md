# Network

## Overview

This document describes the network architecture of the homelab, including VLAN design, firewall strategy and remote access.

The network is built on UniFi (Cloud Gateway + managed switch) with multiple VLANs, strict inter‑VLAN filtering and a self‑hosted WireGuard VPN for remote access.

## VLAN design

The network is segmented into logical VLANs separating management, servers, admin endpoints, Wi‑Fi clients and test workloads.

| VLAN ID | Name        | Subnet           | Purpose |
|--------:|------------|------------------|---------|
| 1       | Management | 10.0.0.0/16      | Management network for UniFi Cloud Gateway and switch. |
| 10      | Servers    | 192.168.10.0/24  | Core infrastructure: Proxmox nodes, TrueNAS, PBS, DNS, reverse proxy, monitoring, logging, SIEM, etc. |
| 30      | Admin      | 192.168.30.0/24  | Admin workstation(s) with full access to management interfaces. |
| 5       | WiFi LAN   | 192.168.5.0/24   | Main Wi‑Fi network (AP + extender, no VLAN tagging, home/guest/IoT devices). |
| 20      | Test       | 192.168.20.0/24  | Isolated test / backup network with very limited access. |
| VPN     | Wireguard  | 192.168.100.0/24 | Virtual private network users. |
<img width="1354" height="265" alt="image" src="https://github.com/user-attachments/assets/63dcd626-3c16-4617-8051-fa286d3f3047" />

## Switching and Wi‑Fi

- Ports connected to Wi‑Fi access points are configured as untagged access ports in VLAN 5.  
- All wireless clients are placed in the WiFi LAN segment and treated as potentially untrusted devices by default.


## Firewall strategy

The firewall on UniFi Cloud Gateway follows a default‑deny, zero‑trust‑inspired strategy between network segments.

Key ideas:

- No implicit trust between VLANs, even inside the home network.  
- Only required flows are explicitly allowed (DNS, HTTPS via reverse proxy, VPN, management from admin VLAN).  
- Catch‑all deny rules at the end of each direction to avoid accidental exposure.

### Example policies

**From internal/server networks to Wi‑Fi / test segments:**

- Allow ICMP for basic diagnostics (ping).  
- Allow selected TCP/UDP ports where needed for testing.  
- Block all 

**From Wi‑Fi / test segments towards internal infrastructure:**

- Allow DNS to internal AdGuard instances.  
- Allow HTTPS to the reverse proxy (Nginx Proxy Manager) for published services.    
- Block all

<img width="1597" height="722" alt="image" src="https://github.com/user-attachments/assets/ca535ddc-cc0e-4f47-b199-66975fba8b71" />

## DNS and name resolution

DNS plays a central role in both usability and security.

- Clients receive AdGuard Home instances as their DNS servers via DHCP.  
- AdGuard filters ads, tracking and known malicious domains, and forwards allowed queries to secure upstream resolvers (e.g. DoH providers).  

<img width="1073" height="648" alt="image" src="https://github.com/user-attachments/assets/0af69a6d-b958-4b69-841f-3b452abf79cd" />
<img width="1064" height="385" alt="image" src="https://github.com/user-attachments/assets/48e99e7c-4e7d-4329-ac2a-bda0d1cbc5be" />

## Remote access (WireGuard + Cloudflare)

Remote access to the homelab is provided by WireGuard running directly on the UniFi Cloud Gateway.

- WireGuard server listens on the router and provides access to internal resources over an encrypted tunnel.  
- A Cloudflare Dynamic DNS setup keeps a hostname updated with the current public IP (no static IP required).  
- No ports other than WireGuard are exposed directly; management interfaces are only reachable once connected via VPN.

<img width="898" height="588" alt="Screenshot 2026-03-23 183611" src="https://github.com/user-attachments/assets/949527a3-9c18-4272-ae60-d4bf2f7f7882" />
<img width="1297" height="348" alt="image" src="https://github.com/user-attachments/assets/ae437b68-5aee-408f-b582-5d3b70720464" />

## Management surface

Only specific networks and devices can reach critical management interfaces.

- Proxmox, TrueNAS, network equipment, Graylog, Wazuh, backup server, etc. are only reachable from the Admin VLAN and selected management hosts.  
- Browser‑based GUIs are exposed internally or via Nginx Proxy Manager with access control lists (ACLs).  
- No direct management access is possible from Wi‑Fi or test networks without going through VPN and/or ACL checks.

<img width="1336" height="258" alt="image" src="https://github.com/user-attachments/assets/20dd31ad-af1c-4861-8002-a54eb76736d0" />
<img width="1331" height="419" alt="image" src="https://github.com/user-attachments/assets/b205f489-d8c5-4966-8db9-fc8668b6fe7d" />

---
