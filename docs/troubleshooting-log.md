# Troubleshooting Log

## Issue: Download from URL failed with a DNS error

**Symptom:** Attempting to download the Ubuntu 24.04.4 Server ISO via Proxmox's Download from URL feature failed with a DNS error.

**Diagnosis:** The real cause wasn't DNS at all. Proxmox's `vmbr0` bridge only had a static IPv6 config — no IPv4 config existed, so the host had zero IPv4 routing. Any IPv4 traffic, DNS lookups included, had nowhere to go.

**Fix:**
1. Edited `/etc/network/interfaces` and added `iface vmbr0 inet dhcp`, giving `vmbr0` IPv4 via DHCP alongside the existing IPv6 config (dual-stack)
2. Converted that IPv4 assignment from DHCP to a static address (with the matching gateway), so the Proxmox web UI link stays constant going forward

---

## Issue: IPv4 still looked broken after the vmbr0 fix

**Symptom:** After fixing `vmbr0`, `ping -4 releases.ubuntu.com` showed 100% packet loss, suggesting IPv4 was still not working.

**Diagnosis — tested in layers, closest to farthest:**
1. `ping -4 <gateway IP>` → succeeded → local network to the router was fine
2. `ping -4 8.8.8.8` (known-good IPv4 address, no DNS lookup involved) → succeeded → router/ISP and general IPv4 routing were fine
3. `curl -4 -I https://releases.ubuntu.com` → returned `HTTP/1.1 200 OK` → the actual protocol that matters (HTTPS) worked perfectly

**Conclusion:** IPv4 connectivity was never actually broken after the `vmbr0` fix. That specific Canonical server (or a hop along the route to it) simply doesn't respond to ICMP (ping), which is common practice for servers/CDNs and unrelated to whether the real connection works. The ping test gave a false negative.

**Lesson:** When ping disagrees with other evidence, don't trust it alone — confirm with the actual protocol you care about. Isolating "is my network broken" from "is this one destination just not responding to this one protocol" via layered testing (nearest hop → known-good external target → actual protocol) is a repeatable diagnostic pattern, not a one-off fix.

## 2026-09-20 - apt update fails to resolve every hostname

Symptom: apt update returned "Temporary failure resolving" for every repo,
including Debian's own.
Cause: /etc/resolv.conf had a single IPv6-only nameserver, left over from the
original IPv6-only vmbr0 config. After switching to static IPv4, nothing supplied
an IPv4 DNS server. Third problem traced to that same vmbr0 config.
Fix: Datacenter -> pve -> System -> DNS, DNS 1 = 192.168.12.1, DNS 2 = 1.1.1.1.
Set in the web UI so it survives reboots.
Follow-up: first try had 198.162.12.1 (transposed digits, a public address). It
worked by failing over to 1.1.1.1, slowly. Corrected to 192.168.12.1.
Takeaway: 192.168.x.x is private, 198.x.x.x is public.

## 2026-09-20 - Console paste inserts garbage

Symptom: pastes came through as $'\E[200~cat...' with a trailing ~.
Cause: noVNC passes bracketed-paste markers through literally.
Fix: use SSH instead. For the console: `bind 'set enable-bracketed-paste off'`.
