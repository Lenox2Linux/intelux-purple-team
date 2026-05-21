# INTELUX Build Phases

A chronological index of the INTELUX build journey.

---

## Phase 0 — Origins (February 2026)
**[→ Full narrative](phases/phase-0-origins.md)**

Single Mac Mini running a flat Docker stack. No segmentation, no domain, no clusters. The starting point.

Key work: Docker stack stabilization, Wazuh initial deployment, Network+ study, first LinkedIn/portfolio assets.

---

## Phase 1 — Expanding the Stack (Late February — Early March 2026)
**[→ Full narrative](phases/phase-1-expanding-stack.md)**

New services deployed, first VMs provisioned, Tailscale remote access established. PhishTix recovered and run locally for the first time.

Key work: Pi-hole, Vaultwarden, Nextcloud, Greenbone, first victim VMs, Tailscale network.

---

## Phase 2 — Naming, VLAN Migration & Wazuh Rebuild (March — April 2026)
**[→ Full narrative](phases/phase-2-naming-vlan-wazuh.md)**

Full naming standardization sprint, VLAN migration from flat network to 10.100.x.x segmented architecture, pfSense bare-metal migration, GRC program established, DC-01 deployed.

Key work: OPNsense battles, pfSense CE build, VLAN migration, Active Directory, GRC artifacts, Kill Chain A prep.

---

## Phase 3 — Mini Rack Build (May 2026)
**[→ Full narrative](phases/phase-3-mini-rack.md)**

9-node physical mini rack acquired and deployed. Two Proxmox clusters built. Full VM rebuild on new infrastructure. Wazuh v4.14.5 deployed with 10 agents.

Key work: Hardware procurement, I219-LM NIC fix, cluster formation, VM rebuild, Wazuh agent enrollment.

---

## Phase 4 — Kill Chain A (May 17, 2026)
**[→ Full narrative](phases/phase-4-kill-chain-a.md)**

First fully authorized red team exercise. RDP initial access, lateral movement attempt, domain pivot — with Wazuh detection as the primary objective. INC-001 filed.

Key work: Pre-flight checklist, RDP exploitation, FIM detection, 400+ Wazuh alerts, post-exercise rollback.

---

## What's Next

- Kill Chain B — outside attacker, reverse shell, no inbound firewall rule
- Suricata IDS on FW-01
- Wazuh agent upgrades on WKS-03 and WKS-04
- VLAN 60 DMZ
- East-west Docker isolation
