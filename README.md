# Home Lab

My cybersecurity home lab — built for hands-on practice with defensive security concepts, networking fundamentals, and detection engineering. This repo documents the build, the learning, and what I'm working on next.

## 🖥️ Hardware

- **UniFi Express** — gateway, router, and network controller
- **UniFi USW Lite 8 PoE** — managed Layer 2 switch (VLAN-capable)
- **UniFi U6+** — wireless access point
- **HP ProDesk 405 G4** — Proxmox VE host

## 🌐 Network

Segmented into isolated zones using 802.1Q VLANs on the UniFi controller and trunked down to the Proxmox host:

| Segment | VLAN ID | Subnet | Purpose |
|---|---|---|---|
| Default (Production) | 1 | 192.168.1.0/24 | Home network, Proxmox management, Tailscale |
| Lab WAN | 66 | 192.168.2.0/24 | Isolated uplink for the virtual firewall |
| Lab LAN (Sandbox) | — | 192.168.10.0/24 | Internal-only network behind OPNsense |

**Isolation controls:**
- UniFi "Isolate Network" rule blocks Layer 3 routing between the Lab VLAN and production
- DHCP Guarding enabled on VLAN 66 to block rogue DHCP servers
- Lab LAN sits on an internal-only Proxmox bridge with no physical uplink

**Remote access:** Tailscale deployed on the Proxmox host for secure management from outside the network.

## 💻 Virtual Machines

| VMID | Name | OS | Role |
|---|---|---|---|
| 101 | kali | Kali Linux | Offensive tooling / scanning practice |
| 102 | opnsense | OPNsense 26.1.6 | Virtual firewall / lab gateway |

## 🧪 Activities So Far

- Stood up Proxmox host and deployed Kali VM
- Configured Tailscale for secure remote access
- Used SSH to manage Proxmox and Kali remotely
- Ran initial nmap scan against my own UniFi gateway to identify exposed services — confirmed perimeter ports closed by default
- **Designed and deployed VLAN segmentation** — created an isolated Lab VLAN (66) with full Layer 3 isolation from production
- **Configured switch port trunking** on the USW Lite 8 PoE to carry tagged VLAN 66 traffic alongside untagged production traffic
- **Deployed OPNsense** as a virtualized firewall with its WAN on the isolated VLAN and its LAN on an internal-only Proxmox bridge
- **Diagnosed and recovered from a production network outage** caused by an early misconfiguration — documented the root cause, recovery steps, and architectural changes that prevent recurrence
- See the full writeup: [OPNsense Virtual Firewall with VLAN-Isolated Lab Network](activities/01-opnsense-vlan-segmentation.md)

## 🗺️ Roadmap

- [x] Design VLAN scheme for network segmentation
- [x] Configure VLANs in UniFi controller
- [x] Deploy virtual firewall (OPNsense) for inter-VLAN routing and firewall learning
- [ ] Build a helper VM on the sandbox LAN to access the OPNsense web UI and begin writing firewall rules
- [ ] Add Ubuntu Server VM (DNS, DHCP, web services)
- [ ] Add Windows 10 client VM
- [ ] Build Active Directory lab (Windows Server + clients)
- [ ] Deploy SIEM stack (Wazuh or Security Onion)
- [ ] Document detection use cases as I build them

## 🎯 Goals

This lab supports my path toward a SOC analyst / blue team role. Specific learning objectives:

- Network segmentation and defense-in-depth
- Firewall rule design and inter-VLAN policy
- SIEM deployment, log analysis, and detection engineering
- Windows + Linux server administration
- Active Directory security
- Hands-on practice with MITRE ATT&CK techniques in a controlled environment

## 📚 Related

- [soc-analyst-notes](https://github.com/Ghostfam44/soc-analyst-notes) — running study notes that complement this build

---

*Work in progress. This lab is evolving as I learn.*
