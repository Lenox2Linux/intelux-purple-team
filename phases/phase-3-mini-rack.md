# Phase 3 — Mini Rack Build (May 2026)

## Hardware Acquisition

Phase 3 began with a hardware procurement run that transformed INTELUX from a single-node Proxmox setup into a 9-node physical mini rack. Hardware acquired via Facebook Marketplace, Mercari, and eBay at significantly below retail cost.

**Hardware acquired:**
- 6x Lenovo ThinkCentre M900 Tiny — Proxmox nodes (BLU-01/02/03, RED-02/03, FW-01)
- 1x Lenovo ThinkCentre M920q — RED-01, primary RED cluster node (32GB RAM)
- 1x AceMagic AM08 Pro (Ryzen 7 6800H, 32GB) — ADM-01 admin workstation replacement
- Rack, patch panel, cables, power distribution

---

## The I219-LM NIC Problem

All 6 M900/M920q nodes use Intel I219-LM network cards. Under network load, this NIC triggers a hardware unit hang — the NIC stops responding entirely and the node drops off the network.

**Root cause:** The I219-LM has a known bug with TCP/UDP segmentation offloading (TSO/GSO/GRO) under certain conditions.

**Fix:** Disable all hardware offloading via `ethtool` and make it persistent via udev rules. Applied to all 6 nodes on first boot.

NIC interface names vary:
- BLU-01/02/03 and RED-02/03: `enp0s31f6`
- RED-01 (M920q): `eno2`

The udev rule must reference the correct interface name per node.

> ⚡ **LESSON:** Preemptively disable TSO/GSO/GRO on all I219-LM nodes via udev rule. Do not wait for the hang to appear under load.

---

## Cluster Architecture

Two Proxmox clusters were created:

**INTELUX-BLUE** — Hosts defender infrastructure
| Node | Hardware | RAM |
|------|----------|-----|
| BLU-01 | M900 Tiny | 32GB |
| BLU-02 | M900 Tiny | 8GB |
| BLU-03 | M900 Tiny | 8GB |

**INTELUX-RED** — Hosts attacker platform, isolated on VLAN 40
| Node | Hardware | RAM |
|------|----------|-----|
| RED-01 | M920q | 32GB |
| RED-02 | M900 Tiny | 8GB |
| RED-03 | M900 Tiny | 8GB |

**CPU Baseline Challenge**

RED cluster mixes M920q (Intel i5-8500T) with M900 (Intel i5-6500T). Both support `x86-64-v2-AES`. VM live migration works across both generations with `x86-64-v2-AES` set as the CPU type.

> ⚡ **LESSON:** Mixed-CPU Proxmox clusters require a common CPU baseline. Set `x86-64-v2-AES` at VM creation time, not after.

---

## VM Build — Phase 3

All VMs rebuilt fresh on the new cluster:

| VM | Host | OS | Role |
|----|------|----|------|
| SVC-01 | BLU-01 | Debian 12 | Wazuh v4.14.5 bare-metal |
| DC-01 | BLU-01 | Windows Server 2022 | INTELUX.local domain controller |
| WKS-03 | BLU-02 | Windows 10 Pro | Domain-joined victim workstation |
| WKS-04 | BLU-03 | Windows 10 Pro | Domain-joined victim workstation |
| KLI-01 | RED-01 | Kali Linux | Primary attack platform |

---

## Wazuh v4.14.5 — 10 Agents

Wazuh v4.14.5 deployed on SVC-01. 10 agents enrolled across the full lab.

| Agent Group | Members |
|-------------|---------|
| default | BLU-01, BLU-02, BLU-03, ADM-01 |
| red-team | RED-01, RED-02, RED-03 |
| CLIENTS | WKS-03, WKS-04 |
| SERVER | DC-01 |

> KLI-01 has no agent — intentional OPSEC decision. The attack platform should not report to the defender's SIEM.

**Notable configuration issue identified:**

`opensearch.yml` nodes_dn CN mismatch — inert on single-node deployment but must be fixed before any scale-out to a second indexer node.

---

## Snapshot Discipline

A critical lesson learned during Phase 3 VM builds:

> ⚡ **LESSON:** RAM-saved snapshots corrupt live network state on restore. Always use disk-only snapshots for network-sensitive VMs. No exceptions.

---

## Phase 3 Key Lessons

| # | Lesson |
|---|--------|
| 1 | I219-LM NIC — preemptively disable TSO/GSO/GRO on all nodes |
| 2 | Mixed-CPU clusters — set CPU baseline at VM creation time |
| 3 | Disk-only snapshots only — never RAM-saved for network VMs |
| 4 | Attack platform intentionally excluded from SIEM (OPSEC) |
| 5 | pfSense VLAN trunk — no native IP on trunk parent (reconfirmed from Phase 2) |
| 6 | opensearch.yml CN mismatch — fix before any scale-out |
