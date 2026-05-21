# INTELUX-BUILD-002 — Suricata IDS + Wazuh Integration

| Field | Value |
|-------|-------|
| **Document ID** | INTELUX-BUILD-002 |
| **Version** | 1.0 |
| **Date** | May 21, 2026 |
| **Author** | Raynard A. Porter — INTELUX Systems |
| **Status** | Final |
| **Classification** | Internal Use Only — Confidential |

---

## 1. Purpose

This document records the installation, configuration, and Wazuh integration of Suricata IDS on INTELUX-FW-01 (pfSense 2.7.2 / FreeBSD 14.0). Suricata runs in IDS-only mode across two interfaces. EVE JSON output is ingested by Wazuh agent 014 and forwarded to the Wazuh manager on SVC-01.

---

## 2. Target System

| Attribute | Value |
|-----------|-------|
| Hostname | INTELUX-FW-01 |
| Role | Core Firewall / pfSense |
| Hardware | Lenovo ThinkCentre M900 |
| OS | pfSense 2.7.2-RELEASE (FreeBSD 14.0-CURRENT) |
| WAN Interface | em0 — MAC `00:23:24:ae:7c:90` |
| LAN Interface | ue0 — MAC `00:05:1b:de:f7:cd` (VLAN trunk) |
| Wazuh Agent ID | 014 |
| Wazuh Agent Version | v4.14.5 |

---

## 3. Suricata Configuration

### 3.1 Interface Summary

| Interface | Description | Blocking Mode | EVE Output | Snaplen |
|-----------|-------------|---------------|------------|---------|
| WAN (em0) | WAN_SURICATA | DISABLED (IDS) | FILE | 1518 |
| LAN (ue0) | LAN_TRUNK | DISABLED (IDS) | FILE | 1522 |

### 3.2 EVE JSON Log Paths

| Interface | Path |
|-----------|------|
| WAN (em0) | `/var/log/suricata/suricata_em042232/eve.json` |
| LAN (ue0) | `/var/log/suricata/suricata_ue01009/eve.json` |

> **Note:** The numeric suffix in the log directory (em042232, ue01009) is assigned at install time. If Suricata is reinstalled, these paths must be re-verified and updated in `ossec.conf`.

### 3.3 Enabled Rulesets

| Ruleset | Source | Update Interval |
|---------|--------|-----------------|
| Emerging Threats Open (ETOpen) | Proofpoint | 12 hours |
| Snort GPLv2 Community Rules | Talos/Cisco | 12 hours |
| Feodo Tracker Botnet C2 IP Rules | abuse.ch | 12 hours |
| ABUSE.ch SSL Blacklist Rules | abuse.ch | 12 hours |

### 3.4 Enabled ET Open Categories (Both Interfaces)

| Category | Purpose |
|----------|---------|
| emerging-attack_response.rules | C2 callbacks and attack tool responses |
| emerging-botcc.rules | Botnet command and control IPs |
| emerging-botcc.portgrouped.rules | Botnet C2 grouped by port |
| emerging-compromised.rules | Known compromised hosts |
| emerging-dns.rules | DNS abuse, tunneling, DGA |
| emerging-dos.rules | Denial of service patterns |
| emerging-drop.rules | Spamhaus DROP list |
| emerging-dshield.rules | DShield top attackers |
| emerging-exploit.rules | Exploit attempts and shellcode |
| emerging-malware.rules | Malware communication patterns |
| emerging-scan.rules | Port and vulnerability scanning |
| emerging-web_server.rules | Web server attacks |

### 3.5 Global Settings

| Setting | Value |
|---------|-------|
| Update Interval | 12 Hours |
| Update Start Time | 00:30 |
| Live Rule Swap on Update | Enabled |
| GeoLite2 DB Update | Disabled |
| Log to System Log | Enabled |
| Keep Settings After Deinstall | Enabled |
| Block Offenders | Disabled (IDS mode) |

---

## 4. Wazuh Agent Installation

### 4.1 Method

Installed via FreeBSD official package repository. FreeBSD repos temporarily enabled on pfSense, package installed, repos re-disabled post-install to prevent pfSense package conflicts.

### 4.2 Installation Commands

```bash
# Enable FreeBSD repos
sed -i '' 's/FreeBSD: { enabled: no/FreeBSD: { enabled: yes/' /usr/local/etc/pkg/repos/FreeBSD.conf
sed -i '' 's/FreeBSD: { enabled: no/FreeBSD: { enabled: yes/' /usr/local/etc/pfSense/pkg/repos/pfSense-repo-*.conf

# Update and install
env IGNORE_OSVERSION=yes pkg update
env IGNORE_OSVERSION=yes pkg install wazuh-agent

# Post-install
cp /etc/localtime /var/ossec/etc
mv /var/ossec/etc/client.keys.sample /var/ossec/etc/client.keys
cp /var/ossec/etc/ossec.conf.sample /var/ossec/etc/ossec.conf

# Enable and start
sysrc wazuh_agent_enable="YES"
service wazuh-agent start

# Re-disable FreeBSD repos
sed -i '' 's/FreeBSD: { enabled: yes/FreeBSD: { enabled: no/' /usr/local/etc/pkg/repos/FreeBSD.conf
sed -i '' 's/FreeBSD: { enabled: yes/FreeBSD: { enabled: no/' /usr/local/etc/pfSense/pkg/repos/pfSense-repo-*.conf

# Register with manager
/var/ossec/bin/agent-auth -m 10.100.20.101 -p 1515
```

### 4.3 ossec.conf — Server Block

```xml
<client>
  <server>
    <address>10.100.20.101</address>
    <port>1514</port>
    <protocol>tcp</protocol>
  </server>
  <config-profile>freebsd, freebsd14</config-profile>
  <crypto_method>aes</crypto_method>
</client>
```

### 4.4 ossec.conf — Suricata EVE Localfile Blocks

Added to end of `/var/ossec/etc/ossec.conf` before `</ossec_config>`:

```xml
<!-- Suricata EVE JSON - WAN -->
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/suricata/suricata_em042232/eve.json</location>
</localfile>

<!-- Suricata EVE JSON - LAN TRUNK -->
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/suricata/suricata_ue01009/eve.json</location>
</localfile>
```

### 4.5 Agent Registration Result

| Attribute | Value |
|-----------|-------|
| Agent ID | 014 |
| Agent Name | INTELUX-FW-01.intelux.local |
| Manager IP | 10.100.20.101 |
| OS Detected | BSD 14.0 |
| Version | v4.14.5 |
| Status | Active |
| Group | default |

---

## 5. Verification

```bash
# Both Suricata processes running
ps aux | grep suricata
# Expected: suricata -i em0 and suricata -i ue0

# WAN EVE writing
cat /var/log/suricata/suricata_em042232/eve.json | head -5

# LAN EVE writing
cat /var/log/suricata/suricata_ue01009/eve.json | head -5

# Wazuh agent log confirming EVE monitoring
tail -30 /var/ossec/logs/ossec.log | grep "Analyzing file"
# Expected: both eve.json paths present

# FreeBSD repos disabled
grep -i enabled /usr/local/etc/pkg/repos/FreeBSD.conf
# Expected: FreeBSD: { enabled: no }
```

Wazuh dashboard confirmed agent 014 **Active** at 14:55 UTC, May 21, 2026.

---

## 6. Known Issues & Deferred Items

| Item | Detail | Priority |
|------|--------|----------|
| IPS mode not enabled | Block Offenders disabled. Baseline IDS for 48-72 hrs, tune suppressions, then evaluate enabling on LAN_TRUNK. | Low — planned |
| Snaplen 1522 not applied on ue0 | Package reports 1518 despite setting 1522. May miss some VLAN-tagged frames. Monitor for missed alerts. | Low |
| No custom Wazuh EVE decoder | Default syslog decoder ingests EVE as raw text. Custom Suricata decoder needed for field-level extraction and alerting. | Medium — next session |
| cis_freebsd14.yml SCA file missing | Rootcheck warns at startup. Non-critical — SCA file not shipped with this package version. | Low |

---

## 7. Next Steps

1. Add custom Suricata EVE JSON decoder to Wazuh on SVC-01
2. Create Wazuh rules for high-severity Suricata categories (exploit, malware, scan)
3. Validate full pipeline — trigger alert from KLI-01, confirm in Wazuh dashboard
4. After 48-72 hr IDS baseline — evaluate enabling IPS on LAN_TRUNK
5. Update Master Asset Inventory → v1.19
6. Proceed to DKR-01 build (next in summer queue)

---

*Document Control: INTELUX-BUILD-002 | Version 1.0 | Classification: Internal — Confidential | Retention: 2 Years*
