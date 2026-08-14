Home SOC Lab:
A home lab SOC/detection environment built on Proxmox. Elastic, Wazuh, and Suricata are deployed manually (not via Security Onion) to build a deeper understanding of each component. The lab is used to run real attacks, observe detection in the SIEM, write custom detection rules, and document each attack-detection-response cycle as a portfolio artifact.


Goal:
Run attacks from a Kali Linux VM (Nmap scans, Metasploit, brute-force attempts, Atomic Red Team) against intentionally vulnerable victim machines, observe detection in the SIEM stack, write custom Suricata and Sigma rules, map findings to MITRE ATT&CK, and document each cycle in this repo.


Architecture:
Proxmox host: Old PC with Intel i5-11400. Dedicated headless hypervisor running all lab VMs.
Client: New PC with Ryzen 7 7800X3D, stays on Windows. Used only to remotely manage the lab via the Proxmox web UI.
Proxmox web UI: reachable at a static internal IP, bookmarked on the client.


Planned VMs:
VM                          Purpose
elastic-vm	                Elasticsearch + Kibana
wazuh-vm	                  Wazuh manager
suricata-vm	                Suricata IDS, positioned to see Kali <-> victim traffic
kali-vm	                    Attacker box
Windows VM	                Victim, with Sysmon for log generation
Metasploitable2 / DVWA	    Deliberately vulnerable victim

Repo structure:
docs/
  progress-log.md         # chronological build log
  troubleshooting-log.md  # problems hit and how they were diagnosed/fixed
attacks/                  # one folder per attack scenario (added as the lab matures)


Status:
Actively in progress. See docs/progress-log.md for current status and docs/troubleshooting-log.md for issues encountered along the way.


Why manual builds instead of Security Onion:
Security Onion bundles this stack together and handles most of the setup automatically. Building Elastic, Wazuh, and Suricata by hand instead is slower, but it means every configuration decision is understood and can be explained in depth, which matters for interview talking points and for actually learning how the pieces fit together.
