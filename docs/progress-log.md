# Progress Log

## Setup

- Two machines: gaming PC (Ryzen 7 7800X3D, RTX 3060 Ti, stays on Windows, used only to remotely manage the lab via browser) and old PC (Intel i5-11400, MSI MAG B560 Torpedo, no GPU, dedicated headless server)
- Old PC runs Proxmox VE, which runs all lab VMs. Gaming PC does no computing for the lab, just remote management via the Proxmox web UI (the "dashboard")
- Proxmox web UI reachable at a static internal IP, bookmarked on the client

## August 2026

**Proxmox install and initial config**
- Installed Proxmox VE on the old PC (i5-11400), wiped the old game SSD in the process
- Resolved a no-display issue post-CPU-swap by clearing CMOS via the JBAT1 jumper
- Confirmed network config (dual NIC board, picked the connected port), set hostname `pve.local`
- Verified in BIOS that boot priority is correctly set to the internal SSD as boot option #1

**Networking fix**
- Attempted to download the Ubuntu 24.04.4 Server ISO via Proxmox's Download from URL feature — failed with a DNS error
- Diagnosed the real cause: `vmbr0` only had a static IPv6 config, no IPv4 at all, so there was zero IPv4 routing (not just a DNS problem)
- Fixed by editing `/etc/network/interfaces`, adding `iface vmbr0 inet dhcp` for dual-stack IPv4+IPv6
- Converted IPv4 from DHCP to a static address so the web UI link stays constant
- Full diagnosis details in `troubleshooting-log.md`

**ISO download**
- Retried the Ubuntu 24.04.4 LTS Server ISO download — completed successfully (`TASK OK`)
- File saved to `/var/lib/vz/template/iso/ubuntu-24.04.4-live-server-amd64.iso`

**GitHub repo setup**
- Created `Home-SOC-Lab` repo on GitHub (KlaudioZen), with description, README, `.gitignore` (None template), and MIT license
- Cloned locally to the gaming PC, configured SSH authentication instead of HTTPS
- Confirmed full clone → pull → commit → push loop works

## Not yet done

- Switch Proxmox repo from "enterprise" to "no-subscription" so `apt update` works without a paid subscription
- Decide RAM/core allocation across `elastic-vm`, `wazuh-vm`, and `suricata-vm`
- Build `elastic-vm` (Elasticsearch + Kibana), `wazuh-vm` (Wazuh manager), `suricata-vm` (Suricata) off the downloaded ISO
- Install Elastic, Wazuh, and Suricata manually inside their respective VMs (decided against Security Onion, want the manual build for interview talking points)
- Build out `kali-vm` (attacker box), Windows VM with Sysmon (victim, log generation), Metasploitable2 or DVWA (deliberately vulnerable victim)
- Later/lower priority: general homelab extras (Pi-hole, WireGuard/Tailscale, etc.)

## Immediate next step

Switch the Proxmox repo to no-subscription, then start building `elastic-vm`.

September 2026
Switched to the no-subscription repo
* Confirmed PVE 9.2.2 on Debian 13 trixie (`pveversion`); this version uses the deb822 `.sources` repo format, not the old one-line `.list` format
* Disabled the paid repos by setting `Enabled: false` on `pve-enterprise.sources` and `ceph.sources`
* Added `pve-no-subscription.sources` pointing at `download.proxmox.com`, with `Signed-By` set to the confirmed keyring path
* `apt update` then failed on every hostname; root cause turned out to be DNS, not the repo edits (full details in `troubleshooting-log.md`)
* After the DNS fix, `apt update` pulled from `pve-no-subscription` cleanly, confirming the repo switch worked

Host upgrade
* Ran `apt full-upgrade`: 182 packages plus a new kernel
* Rebooted into kernel `7.0.14-17-pve`; `pve-manager` now `9.2.20`, `pveproxy` confirmed active

Storage check
* `local-lvm` thin pool has ~794 GB free
* Thin provisioning means a VM disk only consumes what's actually written, so snapshots and clones are cheap. A thin pool that reaches 100% corrupts guests, so watch it with `pvesm status`

## 2026-09-20 - Switched host off the enterprise repo, fixed DNS, upgraded

Switched from the noVNC console to SSH (`ssh root@192.168.12.5`) because the
console mangles pastes. Confirmed PVE 9.2.2 on Debian 13 trixie via `pveversion`
(uses the deb822 .sources repo format).

Disabled the paid repos: appended `Enabled: false` to pve-enterprise.sources and
ceph.sources. Added pve-no-subscription.sources pointing at download.proxmox.com,
Signed-By set to the confirmed keyring path.

apt update then failed on every hostname. Root cause was DNS, not the repo edits
(see troubleshooting log). After the fix, pve-no-subscription appeared in the
output, confirming the repo work.

Ran apt full-upgrade (182 packages + new kernel), rebooted into 7.0.14-17-pve,
pve-manager now 9.2.20, pveproxy active.

Storage: local-lvm thin pool ~794 GB free. Thin provisioning means a 20 GB VM
disk only uses what's written. A thin pool at 100% corrupts guests, so watch it
with `pvesm status`.
