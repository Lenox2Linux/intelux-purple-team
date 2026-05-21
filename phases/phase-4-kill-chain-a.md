# Phase 4 — Kill Chain A: INC-001 (May 17, 2026)

## Exercise Overview

Kill Chain A was the first fully authorized red team exercise executed in the INTELUX lab.

**Scenario:** Insider threat simulation. An attacker with network access to the CLIENTS VLAN initiates an RDP session to a victim workstation using known local admin credentials, then attempts to pivot to the domain controller.

**Authorization:** INTELUX-ROE-001 (Rules of Engagement)

| Item | Value |
|------|-------|
| Attack Platform | KLI-01 (Kali Linux, VLAN 40) |
| Primary Target | WKS-03 (Windows 10 Pro, VLAN 30) |
| Secondary Target | DC-01 (Windows Server 2022, VLAN 30) |
| Detection Objective | Wazuh SIEM — FIM, Windows Event IDs, MITRE ATT&CK alerts |

---

## Pre-Flight Checklist

All 7 ROE checklist items verified before exercise start:

- [x] Wazuh dashboard open — all 10 agents confirmed active
- [x] WKS-03, WKS-04, DC-01 powered on and reachable
- [x] FIM baseline clean — agent.conf deployed, syscheck restarted on target agents
- [x] RED to BLUE pfSense rule enabled
- [x] Disk-only snapshots taken for all target VMs
- [x] INTELUX-ROE-001 reviewed
- [x] INC-001 incident report template open

---

## Attack Timeline

| Time | Action | Result |
|------|--------|--------|
| 22:22 | RED to BLUE pfSense rule enabled. Nmap scan of WKS-03 port 3389 | Port filtered — RDP not open |
| 22:26 | Expanded Nmap scan | Ports 22 and 135 open. RDP still filtered |
| 22:34 | WKS-03 rebooted | Port 3389/tcp OPEN — RDP live |
| 22:34–22:55 | xfreerdp with domain credentials (mwebb) | FAILED — KDC unreachable |
| 22:51 | Fix applied: pointed KLI-01 DNS to DC-01 | INTELUX.local resolving correctly |
| 22:55 | xfreerdp with mwebb domain credentials | FAILED — mwebb has no local admin rights on WKS-03 |
| 22:59 | xfreerdp with local admin account (Intadm) | **SUCCESS — RDP session established** |
| 22:59+ | Session active | ~400 Wazuh alerts and 1,560 FIM hits generated within minutes |

---

## Key Failures & Lessons

**Domain credential failure — DNS was the culprit**

KLI-01 was pointing to a public DNS resolver (1.1.1.1) and could not resolve INTELUX.local. Kerberos requires DNS — if the attack platform cannot resolve the domain name, KDC authentication fails entirely.

Fix: pointed KLI-01 DNS to DC-01.

> ⚡ **LESSON:** Always verify DNS configuration on the attack platform before attempting Kerberos-based authentication.

**mwebb had no local admin rights**

Domain credentials authenticated but mwebb was not in the local Administrators group on WKS-03. Local admin accounts are often the path of least resistance when domain auth fails.

> ⚡ **LESSON:** Have both domain and local admin credentials ready before starting the exercise. Know the privilege level of each.

---

## What Wazuh Detected

### Wazuh Rule IDs Triggered
| Rule ID | Description |
|---------|-------------|
| 550 | FIM — file modified |
| 553 | FIM — file deleted |
| 554 | FIM — file added |
| 594 | Registry key changed |
| 598 | Registry key added |
| 750/751/752 | Registry value changes |

All alerts on agent WKS-03.

### Key FIM Artifacts

**`C:\Windows\System32\spool\drivers\x64\3\tsprint.dll`** — Added on RDP session establishment. This is a Windows print spooler DLL loaded by every RDP session.

> ⚡ **LESSON:** tsprint.dll is a reliable RDP session indicator. When it appears in FIM logs, an RDP session was established. Real SOC analysts flag this pattern.

**Registry activity:**
- `HKLM\System\CurrentControlSet\Services\SharedAccess\Epoch` — modified
- User profile activity under `C:\Users\[local admin account]`

### MITRE ATT&CK Mapping
| Technique | ID | Description |
|-----------|-----|-------------|
| Remote Services: Remote Desktop Protocol | T1021.001 | RDP session from attacker to WKS-03 |
| Valid Accounts: Local Accounts | T1078.003 | Local admin used for initial access |

---

## Post-Exercise Protocol

All post-exercise steps completed in order:

- [x] RED to BLUE pfSense rule disabled immediately after exercise
- [x] WKS-03 and WKS-04 rolled back to pre-exercise snapshots
- [x] Wazuh agents verified active after rollback
- [x] INC-001 filed with full timeline, tools, alerts, FIM changes, findings, and recommendations
- [x] GitHub commit within 48 hours
- [x] Obsidian vault updated
- [x] Master Asset Inventory updated

> ⚡ **LESSON:** Post-exercise rollback is mandatory. Every kill chain exercise must end with target VM rollback and Wazuh agent verification. Document the rollback in the incident report.

> ⚡ **LESSON:** The RED to BLUE pfSense rule is the master kill switch. Enable at exercise start. Disable immediately after. Never leave it enabled between sessions.

---

## Findings & Recommendations

**Finding 1 — RDP Detection via FIM**
tsprint.dll appearance in FIM is a reliable RDP session indicator. Recommend creating a dedicated Wazuh custom rule to alert with HIGH severity on this specific artifact.

**Finding 2 — DNS as a Pre-Auth Dependency**
Attackers operating in segmented environments must verify DNS resolution to the target domain before attempting Kerberos authentication. Blue team should monitor for unusual DNS queries from RED VLAN sources.

**Finding 3 — Local Admin Account Exposure**
The local admin account provided initial access when domain credentials failed. Recommend implementing LAPS (Local Administrator Password Solution) on domain workstations to randomize local admin credentials per machine.

---

## Kill Chain A Status

| Phase | Status |
|-------|--------|
| Initial Access — RDP via local admin | ✅ Complete |
| Persistence | 🔄 Not executed in this exercise |
| Lateral Movement to DC-01 | 🔄 Planned for Kill Chain A Phase 2 |
| Detection & Response | ✅ Complete — INC-001 filed |
