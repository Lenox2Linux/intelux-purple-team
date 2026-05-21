# Phase 1 — Expanding the Stack (Late February — Early March 2026)

## New Services Deployed

With the base Docker stack stable, the lab expanded rapidly. Services added in this phase:

**Pi-hole v6.4** — DNS filtering. Required disabling systemd-resolved and setting `FTLCONF_dns_listeningMode='all'`.

**Vaultwarden** — Self-hosted Bitwarden password manager. Self-signed TLS cert via OpenSSL, signups locked post-setup. Deliberate security decision: web UI only, no browser extension (extension attack surface concern).

**Nextcloud** — Multiple attempts. AIO version abandoned due to Let's Encrypt/SSL conflicts with local/Tailscale setup. Standard Docker version deployed successfully.

> ⚡ **LESSON:** Nextcloud AIO is designed for public deployments with valid Let's Encrypt certs. For a local/Tailscale homelab, use standard Nextcloud Docker instead.

**Greenbone vulnerability scanner** — Deployed via dedicated compose file, initial NVT sync completed.

---

## First VMs Provisioned

Two VMs were provisioned on the primary hypervisor:

- **core1** — Headless Debian 12 VM for general lab use
- **research-01** — Parrot OS 7.1 research VM, configured with Tor Browser and tool separation strategy (Tor for anonymous sessions, LibreWolf + ProtonVPN for account-based research)

**First victim VMs** — Windows 10 VMs configured as deliberate targets:
- SMBv1 enabled
- Windows Defender disabled
- Weak local credentials
- RDP enabled
- IIS FTP with planted credentials

These became the foundation for Kill Chain A in Phase 4.

---

## PhishTix

PhishTix — a Flask/Python phishing analysis and training tool — was recovered from GitHub and successfully run locally. The tool includes hardcoded phishing demo cases with full indicator data, guided analysis questions, and analyst notes.

**Repo:** [github.com/Lenox2Linux/phishtix](https://github.com/Lenox2Linux/phishtix)

---

## Tailscale & Remote Access

Tailscale was configured across all lab devices for remote access. All lab services became reachable via Tailscale IPs from anywhere.

Deliberate decision: victim machines were kept off Tailscale. Attack targets should not have outbound management tunnels — it would compromise lab integrity and realism.

---

## Phase 1 Key Lessons

| # | Lesson |
|---|--------|
| 1 | Nextcloud AIO requires valid public certs — use standard Docker for local/VPN-only deployments |
| 2 | Victim VMs should be isolated from management networks by design |
| 3 | Tool separation matters — different tools for different operational contexts |
