# Home Lab

My cybersecurity home lab — built for hands-on practice with 
defensive security concepts, networking fundamentals, and 
detection engineering. This repo documents the build, the 
learning, and what I'm working on next.

## 🖥️ Hardware

- **UniFi Cloud Gateway** — router / firewall / network controller
- **UniFi PoE Switch** — managed switch (VLAN-capable)
- **UniFi Access Point** — Wi-Fi
- **Proxmox host** — virtualization platform for lab VMs

## 🌐 Network

- Currently a single flat network (default VLAN)
- Managed centrally through the UniFi Network Controller
- **Tailscale** deployed for secure remote access to Proxmox 
  from outside the network

## 💻 Virtual Machines

| VM | OS | Purpose |
|---|---|---|
| kali-01 | Kali Linux | Offensive tooling / scanning practice |

More VMs to come as the lab grows (see roadmap).

## 🧪 Activities So Far

- Stood up Proxmox host and deployed Kali VM
- Configured Tailscale for secure remote access
- Used SSH to manage Proxmox and Kali remotely
- Ran initial **nmap scan** against my own UniFi gateway 
  to identify exposed services — confirmed perimeter ports 
  closed by default
- Began documenting the build (this repo)

## 🗺️ Roadmap

- [ ] Design VLAN scheme for network segmentation
- [ ] Configure VLANs in UniFi controller 
      (trusted / lab / vulnerable / management)
- [ ] Install **pfSense** as a VM in Proxmox for inter-VLAN 
      firewalling and deeper firewall learning
- [ ] Add Ubuntu Server VM (DNS, DHCP, web services)
- [ ] Add Windows 10 client VM
- [ ] Build Active Directory lab (Windows Server + clients)
- [ ] Deploy SIEM stack (Wazuh or Security Onion)
- [ ] Document detection use cases as I build them

## 🎯 Goals

This lab supports my path toward a SOC analyst / blue team role. 
Specific learning objectives:

- Network segmentation and defense-in-depth
- Firewall rule design and inter-VLAN policy
- SIEM deployment, log analysis, and detection engineering
- Windows + Linux server administration
- Active Directory security
- Hands-on practice with MITRE ATT&CK techniques in a 
  controlled environment

## 📚 Related

- [soc-analyst-notes](link-to-your-other-repo) — running study 
  notes that complement this build

---

*Work in progress. This lab is evolving as I learn.*
