# INTELUX-ROE-001 — Rules of Engagement

**Document:** INTELUX-ROE-001  
**Version:** 1.0  
**Applies To:** Kill Chain A, Kill Chain B  
**Status:** Active  

---

## Purpose

This Rules of Engagement document establishes the authorized scope, boundaries, tools, and procedures for all red team exercises conducted within the INTELUX lab environment. All exercises must be authorized under this ROE before commencement.

---

## Authorized Scope

### In Scope
- INTELUX-RED cluster (VLAN 40) as attack origin
- INTELUX-BLUE cluster targets (VLAN 30 — CLIENTS) as designated targets
- WKS-03 and WKS-04 as designated victim workstations
- DC-01 as secondary pivot target (authorized lateral movement only)

### Out of Scope
- FW-01 (pfSense firewall) — no attacks against the firewall itself
- SVC-01 (Wazuh SIEM) — no attacks against the detection infrastructure
- ADM-01 (admin workstation) — no attacks against management systems
- Any home network devices outside the INTELUX VLAN architecture
- Any production systems, cloud services, or external networks

---

## Authorized Tools

| Tool | Purpose |
|------|---------|
| Nmap | Reconnaissance and port scanning |
| xfreerdp | RDP session establishment |
| Metasploit | Exploitation framework (Kill Chain B) |
| Mimikatz | Credential harvesting (future exercises) |
| Kali Linux built-in tools | General attack tooling |

---

## Pre-Flight Checklist

All items must be verified before any exercise begins:

- [ ] Wazuh dashboard open — all relevant agents confirmed active
- [ ] Target VMs powered on and reachable via ping
- [ ] FIM baseline clean — no pre-existing anomalies
- [ ] RED to BLUE pfSense rule confirmed disabled (enable only at exercise start)
- [ ] Disk-only snapshots taken for all target VMs
- [ ] ROE reviewed and confirmed
- [ ] Incident report template open and ready

---

## Exercise Boundaries

**Network isolation:** The RED cluster operates exclusively on VLAN 40. The RED to BLUE inter-VLAN pfSense rule is the master kill switch and must be:
- Enabled only at the moment the exercise begins
- Disabled immediately upon exercise completion
- Never left enabled between sessions

**Time boundaries:** Exercises should be conducted during scheduled lab sessions only. No unattended attack automation.

**Credential policy:** Only lab-created credentials are used. No real-world credentials, no external accounts.

---

## Post-Exercise Protocol

Upon completion of every exercise, the following steps must be completed in order:

1. Disable RED to BLUE pfSense rule immediately
2. Roll back all target VMs to pre-exercise snapshots
3. Verify all Wazuh agents active after rollback
4. File incident report with full timeline, tools, alerts, FIM changes, findings, and recommendations
5. GitHub commit within 48 hours
6. Update Obsidian vault
7. Update Master Asset Inventory

---

## Incident Reporting

All exercises generate a formal incident report using the INTELUX incident report template.

| Report | Exercise |
|--------|---------|
| INC-001 | Kill Chain A — RDP Initial Access |
| INC-002+ | Future exercises |

---

## Authorization

This ROE is authorized for all exercises conducted within the INTELUX lab environment by the lab owner and operator.

**Lab Owner:** Raynard A. Porter  
**Environment:** INTELUX Enterprise Homelab (private, sandboxed, non-production)  
**Authorization Date:** May 2026
