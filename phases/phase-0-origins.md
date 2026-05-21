# Phase 0 — Origins (February 2026)

## The Starting Point

The INTELUX project began in February 2026 with a single Mac Mini running Ubuntu Server and a collection of Docker containers. At this stage the lab was essentially a flat Docker stack on one machine — Wazuh SIEM, Grafana, Prometheus, Uptime Kuma, and a Homepage dashboard, all running on the same host with no network segmentation.

The network was a dual-router setup: a primary home gateway and a dedicated lab router connected via Ethernet. A Netgear GS308E managed switch sat behind the lab router. The lab was really just a slightly organized collection of services on one machine.

---

## Early Infrastructure Work

The first real technical sessions focused on getting the Docker stack stable. Key early challenges:

**Grafana volume mount permission issues** — fixed by running Grafana without a volume mount until the correct UID/GID ownership was established.

**Prometheus node exporter on Proxmox** — enterprise repos blocked apt install. Required downloading binaries from GitHub and creating systemd service units manually.

**Wazuh configuration crash loop** — root cause was malformed XML in the bind-mounted manager config.

> ⚡ **LESSON:** Editing the Docker volume directly is ineffective. The host-side bind-mount source file must be edited, then `docker compose down` / volume removal / `docker compose up`. The container init script overwrites the volume from source on every restart.

**Multi-channel alerting** — Postfix Gmail relay configured for email alerts. Telegram alerting via custom Python integration script.

---

## Network+ Studies

Running in parallel with the lab build, Network+ exam preparation was active. Identified weak areas at this phase:

- CIDR summarization — consistently one prefix step short
- IPv6 address structure
- Subnet mask calculation
- ARP protocol scope

These weaknesses became directly relevant when designing the INTELUX VLAN architecture in later phases.

---

## Portfolio Foundation

Alongside the technical work, early sessions produced the first INTELUX portfolio assets — professional banner, network topology graphics, and LinkedIn post copy. This set the tone for INTELUX as a documented, portfolio-quality project from the beginning.

---

## Phase 0 Key Lessons

| # | Lesson |
|---|--------|
| 1 | Always edit the bind-mount source on the host, never inside the container |
| 2 | Document every failure as thoroughly as every success — the failures teach more |
| 3 | A portfolio-quality lab requires portfolio-quality documentation from day one |
