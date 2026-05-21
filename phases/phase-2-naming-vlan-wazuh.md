# Phase 2 — Naming, VLAN Migration & Wazuh Rebuild (March — April 2026)

## The Naming Standardization Sprint

The lab had accumulated organic, inconsistent hostnames. For a portfolio-quality enterprise environment, this was unacceptable. A full naming standardization was executed in a single extended session — every host, every VM, every Wazuh agent renamed to follow the INTELUX convention.

**The Wazuh Agent Rename Challenge**

The Wazuh web UI does not support direct agent renaming. The solution was using SQLite3 executed inside the Docker container against the global database. Each agent name was updated directly in the database.

---

## The OPNsense Wars

The INTELUX network architecture called for proper enterprise-grade segmentation: five VLANs on 10.100.x.x subnets, managed by a dedicated firewall VM. OPNsense was the first choice.

**Battle 1: The Single NIC Problem**

The primary Proxmox host had only one physical NIC. OPNsense needs separate WAN and LAN interfaces. Single-NIC trunk mode is significantly more complex to troubleshoot and created a dependency loop — to configure OPNsense LAN you need to reach it, but to reach it you need it configured. Every misstep locked out access entirely.

**Battle 2: The GS308E Config-Save Bug**

The Netgear GS308E does not auto-save configuration changes. Every VLAN change must be followed by **System → Maintenance → Save Configuration** — otherwise the configuration is lost on reboot. Discovered the hard way, multiple times.

> ⚡ **LESSON:** GS308E — ALWAYS use System → Maintenance → Save Configuration after EVERY change. No exceptions.

**Battle 3: The Network Loop**

An attempt to add redundant uplinks between switches created a network loop. The GS308E does not support STP (Spanning Tree Protocol). The loop brought down the entire lab network.

> ⚡ **LESSON:** GS308E does not support STP. Never connect two ports of the same switch together — it creates a loop.

**Battle 4: Kea DHCP**

OPNsense's Kea DHCP server refused to issue leases. Fix: changing the socket type from UDP to RAW in Kea configuration.

> ⚡ **LESSON:** Kea DHCP on OPNsense requires socket type RAW, not UDP.

**Battle 5: The GS308E Factory Reset Trap**

Recovering a locked-out GS308E required a factory reset. If the switch is connected to any other device during the reset, the connected router's DHCP server overwrites the switch's default management IP before access is possible. The switch must be completely physically isolated for the factory reset to hold.

> ⚡ **LESSON:** GS308E factory reset requires complete physical isolation from all other devices.

**The Decision**

After multiple sessions fighting OPNsense — interface mismatch warnings, DHCP failures, and access lockouts — the decision was made to switch to pfSense CE. A comprehensive OPNsense runbook was produced documenting all failure points for future reference.

> ⚡ **LESSON:** When a tool has cost multiple full sessions without progress, document what you learned and switch tools. Time is the most finite resource.

---

## The pfSense Build

pfSense CE was installed fresh on a dedicated VM. The single-NIC problem was solved using a USB Ethernet adapter (Realtek RTL8153).

**The RTL8153 Driver Battle**

The USB NIC was detected but never stayed up. The driver loaded but was immediately reset by `r8152-cfgselector` in a loop.

Fix: Blacklisting `r8152-cfgselector` by creating `/etc/modprobe.d/r8152-fix.conf`. After a full system reboot, the USB NIC appeared and stayed up.

> ⚡ **LESSON:** Never install firmware packages that would remove `proxmox-ve` or `proxmox-kernel` as dependencies. It destroys the Proxmox installation.

**VLAN Configuration**

All 5 VLANs configured in pfSense:

| VLAN | Subnet | Purpose |
|------|--------|---------|
| VLAN 10 — MGMT | 10.100.10.0/24 | Management |
| VLAN 20 — SERVICES | 10.100.20.0/24 | Internal services |
| VLAN 30 — CLIENTS | 10.100.30.0/24 | Domain workstations |
| VLAN 40 — RED | 10.100.40.0/24 | Attacker network |
| VLAN 50 — STAGING | 10.100.50.0/24 | Future use |

---

## Active Directory: DC-01 Deployed

DC-01 was deployed as a Windows Server 2022 domain controller for the **INTELUX.local** domain. This was a significant milestone — the lab now had real enterprise identity infrastructure.

Domain users created for Kill Chain A realism: `mwebb`, `dreyes`, `svc-backup`.

WKS-03 and WKS-04 were domain-joined to INTELUX.local.

**DC-01 VLAN Placement Incident (INC-006)**

DC-01 was initially placed on VLAN 20 (SERVICES). WKS-03 on VLAN 30 (CLIENTS) could not reach it for domain join — pfSense blocks CLIENTS to SERVICES by design. Fix: moved DC-01 to VLAN 30 (CLIENTS) so it is co-located with the machines that authenticate against it.

> ⚡ **LESSON:** Plan VLAN placement around communication requirements before deployment. Domain controllers must be reachable by all client VLANs.

---

## Wazuh Rebuild

The Wazuh deployment moved from the Mac Mini Docker stack to a dedicated Debian 12 VM running Wazuh v4.14.5 as a bare-metal single-node installation. This eliminated Docker overhead and gave Wazuh its own dedicated compute.

> ⚡ **LESSON:** Always verify the shell hostname before running install or uninstall commands. Running an uninstall on the wrong host required a snapshot rollback.

---

## GRC Program Established

The formal GRC program was launched in Phase 2:

- **INTELUX-ROE-001** — Rules of Engagement for Kill Chain A and B
- **RISK-001** — Unrestricted inter-VLAN lateral movement risk, mitigated and closed
- **ISO 27001 compliance matrix** — 28 Annex A controls tracked
- **INC-001 through INC-006** — Incident reports for all detected issues

See [grc/](../grc/) for full GRC artifacts.

---

## The VLAN Migration

After weeks of pfSense configuration, the actual migration was executed — moving every device from the flat home network into its correct VLAN.

Migration order: ADM-01 first (least critical), then SRV-01 (services), then hypervisors (most complex), then RED-01 last.

**pfSense Bare-Metal Migration**

pfSense was originally running as a VM inside HV-01. This created a dependency loop: migrating HV-01 could break pfSense, which would break the entire lab network. Decision: migrate pfSense to bare-metal on a dedicated M900 Tiny. The right architectural call — a firewall should not depend on the hypervisor it protects.

> ⚡ **LESSON:** FreeBSD/pfSense VLAN trunk rule — Do not bind a native IP to the VLAN trunk parent interface. It silently drops all 802.1Q tagged frames.

> ⚡ **LESSON:** Firewall independence — The firewall should not run inside the hypervisor it protects. Bare-metal separation eliminates the dependency loop.

---

## Phase 2 Key Lessons

| # | Lesson |
|---|--------|
| 1 | GS308E — Save config after every change, no exceptions |
| 2 | GS308E does not support STP — no redundant uplinks |
| 3 | Kea DHCP requires RAW socket type |
| 4 | Factory reset GS308E requires physical isolation |
| 5 | Never install packages that remove proxmox-ve as a dependency |
| 6 | RTL8153 fix: blacklist r8152-cfgselector |
| 7 | pfSense VLAN trunk — no native IP on trunk parent |
| 8 | Firewall must be independent of the hypervisor it protects |
| 9 | DC-01 must be on the same VLAN as domain clients |
| 10 | Always verify hostname before running install/uninstall commands |
