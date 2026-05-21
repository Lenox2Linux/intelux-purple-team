# INTELUX Enterprise Homelab

> A portfolio-quality purple team lab built from scratch on consumer hardware.

**Author:** Raynard A. Porter ([@Lenox2Linux](https://github.com/Lenox2Linux))  
**Build Period:** February 2026 — Present  
**Status:** Active — Kill Chain A Complete | Kill Chain B Planned

---

## What Is INTELUX?

INTELUX is a rack-mounted enterprise-grade cybersecurity lab designed for real purple team operations — not simulated exercises, not guided walkthroughs. Real infrastructure, real attacks, real detection.

The lab runs a full enterprise stack: Active Directory domain, VLAN-segmented network, pfSense firewall, Proxmox hypervisor clusters, and Wazuh SIEM monitoring every endpoint. The RED cluster attacks. The BLUE cluster defends. Wazuh watches everything.

This repo documents the full build journey — every victory, every failure, and every lesson learned along the way.

---

## Current Architecture

### Physical Infrastructure
| Node | Hardware | Role | Cluster |
|------|----------|------|---------|
| FW-01 | Lenovo M900 Tiny | pfSense CE Firewall (bare-metal) | — |
| SW-01 | Netgear GS308E | Managed Switch | — |
| BLU-01 | Lenovo M900 Tiny (32GB) | Proxmox — Primary BLUE node | INTELUX-BLUE |
| BLU-02 | Lenovo M900 Tiny (8GB) | Proxmox — BLUE node | INTELUX-BLUE |
| BLU-03 | Lenovo M900 Tiny (8GB) | Proxmox — BLUE node | INTELUX-BLUE |
| RED-01 | Lenovo M920q (32GB) | Proxmox — Primary RED node | INTELUX-RED |
| RED-02 | Lenovo M900 Tiny (8GB) | Proxmox — RED node | INTELUX-RED |
| RED-03 | Lenovo M900 Tiny (8GB) | Proxmox — RED node | INTELUX-RED |
| ADM-01 | AceMagic AM08 Pro (32GB) | Admin workstation (Debian 13) | — |

### Virtual Machines
| VM | Host | OS | Role |
|----|------|----|------|
| SVC-01 | BLU-01 | Debian 12 | Wazuh v4.14.5 SIEM |
| DC-01 | BLU-01 | Windows Server 2022 | Domain Controller (INTELUX.local) |
| WKS-03 | BLU-02 | Windows 10 Pro | Domain-joined victim workstation |
| WKS-04 | BLU-03 | Windows 10 Pro | Domain-joined victim workstation |
| KLI-01 | RED-01 | Kali Linux | Primary attack platform |

### Network Segmentation
| VLAN | Name | Purpose |
|------|------|---------|
| VLAN 10 | MGMT | Management — hypervisors, switch, firewall |
| VLAN 20 | SERVICES | Internal services |
| VLAN 30 | CLIENTS | Domain workstations and domain controller |
| VLAN 40 | RED | Isolated attacker network |
| VLAN 50 | STAGING | Future use |

---

## SIEM Coverage

Wazuh v4.14.5 — 11 agents enrolled across all nodes.

| Agent ID | Name | Group | OS |
|----------|------|-------|----|
| 001 | INTELUX-BLU-01 | default | Debian 12 |
| 002 | INTELUX-BLU-02 | default | Debian 12 |
| 003 | INTELUX-BLU-03 | default | Debian 12 |
| 004 | INTELUX-ADM-01 | default | Windows 10 |
| 005 | INTELUX-RED-01 | red-team | Debian 12 |
| 006 | INTELUX-RED-02 | red-team | Debian 12 |
| 007 | INTELUX-RED-03 | red-team | Debian 12 |
| 011 | INTELUX-DC-01 | SERVER | Windows Server 2022 |
| 012 | INTELUX-WKS-03 | CLIENTS | Windows 10 |
| 013 | INTELUX-WKS-04 | CLIENTS | Windows 10 |
| 014 | INTELUX-FW-01 | default | FreeBSD 14.0 (pfSense) |

> KLI-01 has no Wazuh agent — intentional OPSEC decision.

**Suricata IDS** is deployed on FW-01 across WAN (em0) and LAN trunk (ue0) in IDS mode. EVE JSON logs forwarded to Wazuh via agent 014. See [docs/INTELUX-BUILD-002-Suricata-Wazuh-Integration.md](docs/INTELUX-BUILD-002-Suricata-Wazuh-Integration.md).

---

## Kill Chain Status

| Exercise | Status | Incident Report |
|----------|--------|----------------|
| Kill Chain A — RDP Initial Access & FIM Detection | ✅ Complete | INC-001 |
| Kill Chain B — Reverse Shell (Outside Attacker) | 🔄 Planned | — |

---

## Repo Structure

```
intelux-purple-team/
├── README.md                  ← You are here
├── PHASES.md                  ← Phase summary index
├── phases/                    ← Full build narrative by phase
├── grc/                       ← GRC artifacts (ROE, risk register, ISO 27001)
├── lessons/                   ← Master lessons learned
└── docs/                      ← Architecture and reference docs
```

---

## Tools & Stack

`pfSense CE` `Proxmox VE` `Wazuh v4.14.5` `Suricata` `Kali Linux` `Windows Server 2022`  
`Active Directory` `Netgear GS308E` `Debian 12` `xfreerdp` `Nmap` `Wireshark`

---

## Related Projects

- [PhishTix](https://github.com/Lenox2Linux/phishtix) — AI-powered phishing analyst training tool
- [homelab-cmdb](https://github.com/Lenox2Linux/homelab-cmdb) — Enterprise-style CMDB and infrastructure documentation

---

*Built in the Bronx. Documented for the portfolio. Operated like production.*
