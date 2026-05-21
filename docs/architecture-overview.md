# INTELUX Architecture Overview

## Network Architecture

INTELUX uses a fully segmented VLAN architecture managed by pfSense CE on dedicated bare-metal hardware.

```
Internet
    │
[FW-01 — pfSense CE — M900 Tiny]
    │
[SW-01 — Netgear GS308E]
    ├── VLAN 10 — MGMT     (Hypervisors, switch, firewall management)
    ├── VLAN 20 — SERVICES (Internal services)
    ├── VLAN 30 — CLIENTS  (Domain workstations, domain controller)
    ├── VLAN 40 — RED      (Isolated attacker network)
    └── VLAN 50 — STAGING  (Future use)
```

---

## Physical Node Map

| Node | Hardware | VLAN | Role |
|------|----------|------|------|
| FW-01 | Lenovo M900 Tiny | All (trunk) | pfSense CE firewall |
| SW-01 | Netgear GS308E | All | Managed switch |
| BLU-01 | Lenovo M900 Tiny (32GB) | MGMT + CLIENTS | Proxmox — BLUE primary |
| BLU-02 | Lenovo M900 Tiny (8GB) | MGMT + CLIENTS | Proxmox — BLUE node |
| BLU-03 | Lenovo M900 Tiny (8GB) | MGMT + CLIENTS | Proxmox — BLUE node |
| RED-01 | Lenovo M920q (32GB) | MGMT + RED | Proxmox — RED primary |
| RED-02 | Lenovo M900 Tiny (8GB) | MGMT + RED | Proxmox — RED node |
| RED-03 | Lenovo M900 Tiny (8GB) | MGMT + RED | Proxmox — RED node |
| ADM-01 | AceMagic AM08 Pro (32GB) | MGMT | Admin workstation |

---

## VM Map

| VM | Host Node | OS | VLAN | Role |
|----|-----------|-----|------|------|
| SVC-01 | BLU-01 | Debian 12 | SERVICES | Wazuh v4.14.5 SIEM |
| DC-01 | BLU-01 | Windows Server 2022 | CLIENTS | INTELUX.local domain controller |
| WKS-03 | BLU-02 | Windows 10 Pro | CLIENTS | Domain-joined victim workstation |
| WKS-04 | BLU-03 | Windows 10 Pro | CLIENTS | Domain-joined victim workstation |
| KLI-01 | RED-01 | Kali Linux | RED | Primary attack platform |

---

## Proxmox Clusters

**INTELUX-BLUE**
- Members: BLU-01, BLU-02, BLU-03
- Purpose: Defender infrastructure hosting

**INTELUX-RED**
- Members: RED-01, RED-02, RED-03
- Purpose: Attacker platform hosting, isolated on VLAN 40

**CPU Baseline:** `x86-64-v2-AES` — required for live migration across M920q and M900 nodes.

---

## Wazuh Agent Map

| Agent Group | Enrolled Members |
|-------------|-----------------|
| default | BLU-01, BLU-02, BLU-03, ADM-01 |
| red-team | RED-01, RED-02, RED-03 |
| CLIENTS | WKS-03, WKS-04 |
| SERVER | DC-01 |

Total: 10 agents active  
KLI-01: No agent (intentional OPSEC)

---

## Firewall Rules — Key Inter-VLAN Policies

| Rule | Source | Destination | Action | Notes |
|------|--------|-------------|--------|-------|
| RED to BLUE | VLAN 40 | VLAN 30 | BLOCK (default) | Enabled only during authorized exercises |
| CLIENTS to SERVICES | VLAN 30 | VLAN 20 | BLOCK | Prevents client lateral movement to services |
| MGMT full access | VLAN 10 | All | ALLOW | Admin workstation full reach |
| All VLANs internet | Any | WAN | ALLOW | Outbound NAT |

---

## Active Directory

**Domain:** INTELUX.local  
**Domain Controller:** DC-01  
**Domain-joined members:** WKS-03, WKS-04

**Domain users (lab/exercise accounts):**
- `mwebb` — standard domain user, no local admin rights
- `dreyes` — standard domain user
- `svc-backup` — service account

---

## Key Design Decisions

**Firewall on bare-metal** — pfSense runs on dedicated M900 hardware, not as a VM inside a hypervisor it protects. This eliminates the dependency loop where migrating the hypervisor breaks the firewall.

**RED cluster isolation** — RED VMs are physically and logically isolated on VLAN 40. The only path to BLUE infrastructure is the master kill switch inter-VLAN rule in pfSense.

**No Wazuh agent on KLI-01** — The attack platform should not report to the defender's SIEM. OPSEC decision.

**Disk-only snapshots** — RAM-saved snapshots corrupt live network state on restore. All snapshots are disk-only.
