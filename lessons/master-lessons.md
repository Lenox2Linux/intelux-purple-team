# Master Lessons Learned

Every hard lesson from the INTELUX build — Phase 0 through Kill Chain A.

> These are the lessons that cost real time, broke real things, and required real recovery. They are documented here so they are never forgotten and never repeated.

---

## Infrastructure & Proxmox

| ⚡ | Lesson |
|----|--------|
| ⚡ | Single NIC Proxmox hosts cannot run pfSense/OPNsense in standard two-NIC mode without a USB NIC or PCIe card |
| ⚡ | RTL8153 USB NIC fix: blacklist `r8152-cfgselector` in `/etc/modprobe.d/` — requires full reboot |
| ⚡ | VM clones inherit Wazuh agent IDs — delete agent from manager and clear `client.keys` before re-registration |
| ⚡ | BTRFS filesystem can cause chroot password reset failures — consider fresh install for stubborn VMs |
| ⚡ | Never install packages that would remove `proxmox-ve` or `proxmox-kernel` as dependencies |
| ⚡ | `hostnamectl set-hostname` on a Proxmox node affects Proxmox node identity — use with caution |
| ⚡ | Snapshot discipline: RAM-saved snapshots corrupt live network state on restore. Always use disk-only snapshots for network-sensitive VMs. No exceptions. |
| ⚡ | Always verify the shell hostname before running install or uninstall commands |
| ⚡ | I219-LM NIC hardware unit hang: preemptively disable TSO/GSO/GRO via udev rule on all nodes with this NIC |
| ⚡ | Mixed-CPU Proxmox clusters: `x86-64-v2-AES` required for live migration between M920q and M900 nodes. Set at VM creation time, not after. |

---

## Networking & pfSense

| ⚡ | Lesson |
|----|--------|
| ⚡ | GS308E — ALWAYS use System → Maintenance → Save Configuration after EVERY change. No exceptions. |
| ⚡ | GS308E factory reset requires complete physical isolation from all other devices |
| ⚡ | GS308E does not support STP — never create redundant uplinks without STP |
| ⚡ | Kea DHCP on OPNsense/pfSense requires socket type RAW, not UDP |
| ⚡ | pfSense blocks web UI from WAN by default — use a static IP on the LAN subnet for first access |
| ⚡ | pfSense interface mismatch on boot = old interface names in config. Re-assign via console. |
| ⚡ | FreeBSD/pfSense VLAN trunk rule: do not bind a native IP to the VLAN trunk parent interface. It silently drops all 802.1Q tagged frames. All VLANs must be sub-interfaces only. |
| ⚡ | Firewall independence: the firewall should not run inside the hypervisor it protects. Bare-metal separation eliminates the dependency loop. |
| ⚡ | pfSense rule order is top-down. Block rules above allow rules silently drop traffic. Always verify ordering after any rule change. |
| ⚡ | pfSense config backup is a pre-session discipline item. A config backup should exist before any major change. |

---

## VPN & Remote Access

| ⚡ | Lesson |
|----|--------|
| ⚡ | Tailscale + ProtonVPN conflict: set `tailscale --accept-dns=false` to give ProtonVPN full DNS control |
| ⚡ | After `--accept-dns=false`, use Tailscale IPs directly (not MagicDNS hostnames) for SSH |

---

## SIEM & Wazuh

| ⚡ | Lesson |
|----|--------|
| ⚡ | Never edit inside a running Docker container's volume directly. Always edit the host-side bind-mount source. The container overwrites volumes from source on every restart. |
| ⚡ | rsyslog wraps EVE JSON in a syslog envelope that breaks Wazuh's built-in Suricata decoder. Strip the wrapper or write a custom decoder. |
| ⚡ | Wazuh agent clones: clear `client.keys` and re-register before deploying a VM cloned from another enrolled machine |
| ⚡ | `opensearch.yml` nodes_dn CN mismatch is inert on single-node but blocks cluster formation on scale-out |
| ⚡ | `wazuh-passwords-tool.sh` keystore update is unreliable. Always follow with manual `opensearch-dashboards-keystore add --force` |
| ⚡ | Wazuh web UI does not support direct agent renaming — use SQLite3 against the global database |

---

## Active Directory & Identity

| ⚡ | Lesson |
|----|--------|
| ⚡ | Plan VLAN placement around communication requirements before deployment. Domain controllers must be reachable by all client VLANs. |
| ⚡ | Kerberos requires DNS. If the attack platform cannot resolve the domain name, KDC authentication fails. Verify DNS on the attacker before attempting domain credential auth. |

---

## Red Team Operations

| ⚡ | Lesson |
|----|--------|
| ⚡ | tsprint.dll is a reliable RDP session indicator. When it appears in FIM logs, an RDP session was established. |
| ⚡ | Local admin accounts are often the path when domain auth fails. Have both domain and local admin credentials ready before the exercise. |
| ⚡ | Post-exercise rollback is mandatory. Every kill chain exercise must end with target VM rollback and Wazuh agent verification. |
| ⚡ | The RED to BLUE pfSense rule is the master kill switch. Enable at exercise start. Disable immediately after. Never leave it enabled between sessions. |
| ⚡ | Outside attacker realism (Kill Chain B): the victim machine must initiate the outbound connection. No inbound pfSense rule modification. |

---

## Documentation & GRC

| ⚡ | Lesson |
|----|--------|
| ⚡ | Every exercise must produce: incident report, Wazuh alert screenshots, FIM before/after comparison, post-exercise rollback confirmation, GitHub commit within 48 hours |
| ⚡ | Closed risk records are not deleted when the underlying asset changes — retain as GRC artifacts |
| ⚡ | INC numbering must be consistent across all documentation |
| ⚡ | Document every failure as thoroughly as every success — the failures teach more |
| ⚡ | Always know your rollback plan before making changes to production lab infrastructure |

---

## Mindset

| ⚡ | Lesson |
|----|--------|
| ⚡ | When a tool has cost multiple full sessions without progress: document what you learned and switch tools |
| ⚡ | Hardware age matters less than usage profile — old hardware with low hours outlasts newer hardware with high hours |
| ⚡ | A portfolio-quality lab requires portfolio-quality documentation from day one |
