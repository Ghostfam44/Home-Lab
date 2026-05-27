# OPNsense Virtual Firewall with VLAN-Isolated Lab Network

**Date:** May 26, 2026
**Status:** Completed
**Skills demonstrated:** Network segmentation, 802.1Q VLAN tagging, virtualization, firewall deployment, incident response, remote troubleshooting

---

## Objective

Deploy a virtualized firewall (OPNsense) inside Proxmox to act as the gateway for an isolated lab sandbox. The firewall needed real internet access for offensive/defensive testing, but had to be completely walled off from the production home network so that misconfigurations or compromised lab VMs could not impact production devices.

---

## Architecture

### Hardware
- **UniFi Express** — gateway, router, and network controller
- **UniFi USW Lite 8 PoE** — managed Layer 2 switch (VLAN-capable)
- **UniFi U6+** — wireless access point
- **HP ProDesk 405 G4** — Proxmox VE host

### Network Segments
| Segment | VLAN ID | Subnet | Purpose |
|---|---|---|---|
| Default (Production) | 1 | 192.168.1.0/24 | Home network, Proxmox management, Tailscale |
| Lab WAN | 66 | 192.168.2.0/24 | Isolated uplink for the virtual firewall |
| Lab LAN (Sandbox) | — | 192.168.10.0/24 | Internal-only network behind OPNsense |

### Remote Access
- **Tailscale** mesh VPN deployed on the Proxmox host for remote management of the lab from outside the LAN.

---

## Virtual Machines

| VMID | Name | OS | WAN | LAN |
|---|---|---|---|---|
| 101 | kali | Kali Linux | vmbr0 (production) | — |
| 102 | opnsense | OPNsense 26.1.6 | vmbr66 (VLAN 66, isolated) | vmbr1 (internal sandbox) |

---

## Implementation

### 1. VLAN Creation (UniFi)
Created a new virtual network in the UniFi controller:
- **Name:** Lab
- **VLAN ID:** 66
- **Subnet:** 192.168.2.0/24 (gateway 192.168.2.1)
- **Isolate Network:** Enabled — blocks Layer 3 routing between the Lab VLAN and other networks
- **Allow Internet Access:** Enabled
- **DHCP Guarding:** Enabled, with 192.168.2.1 set as the trusted DHCP server (blocks rogue DHCP servers on this VLAN)
- **Ping Conflict Detection:** Enabled

### 2. Switch Port Trunking
Configured the USW Lite 8 PoE switch port feeding the Proxmox host:
- **Port:** 3 (uplink to HP ProDesk)
- **Native VLAN:** Default (1) — preserves untagged management traffic
- **Tagged VLANs:** Lab (66)

This turns Port 3 into a trunk: untagged production traffic and tagged VLAN 66 ride the same physical cable to the Proxmox host.

### 3. Proxmox VLAN Bridge
Created a new Linux bridge on the Proxmox host to terminate VLAN 66:
- **Bridge name:** vmbr66
- **Bridge port:** `enp0s31f6.66` (physical NIC with the `.66` suffix pulls only tagged VLAN 66 traffic)
- **VLAN aware:** disabled (not needed — the `.66` subinterface handles the tag)

Applied network configuration.

### 4. OPNsense VM Deployment
Created VM 102 in Proxmox:
- **CPU:** 2 cores
- **Memory:** 2048 MB
- **Disk:** 32 GB on local-lvm (SCSI, VirtIO SCSI single controller)
- **Network 0 (WAN):** VirtIO on `vmbr66` (isolated VLAN 66)
- **Network 1 (LAN):** VirtIO on `vmbr1` (internal-only bridge, no physical ports)
- **ISO:** OPNsense-26.1.6-dvd-amd64

### 5. OPNsense Installation and Interface Configuration
Installed OPNsense using the UFS installer, then used the console wizard to:
- **Assign interfaces:** WAN → vtnet0, LAN → vtnet1
- **LAN static IP:** 192.168.10.1/24
- **LAN DHCP scope:** 192.168.10.10 – 192.168.10.200
- **WAN:** DHCP client — pulled lease `192.168.2.103/24` from the UniFi Lab VLAN

### 6. Validation
From the OPNsense console, pinged `8.8.8.8`:
```
3 packets transmitted, 3 packets received, 0% packet loss
```
End-to-end path confirmed: OPNsense → vmbr66 → tagged VLAN 66 → switch port 3 trunk → UniFi Express → internet.

---

## Incident: Production Network Outage During Initial Deployment

The first deployment attempt of OPNsense caused a network-wide outage. Documenting this because the diagnostic process is the actual skill being demonstrated.

### What happened
The initial VM build placed the OPNsense WAN interface on `vmbr0` — the same Proxmox bridge used by the host's production management interface and Tailscale connection. When the OPNsense installer booted, it brought up its WAN interface on the live 192.168.1.0/24 network. Within minutes:
- The Proxmox host dropped from Tailscale (last seen 8 minutes prior to first detection)
- UniFi access point and switch began flapping (devices repeatedly disconnecting and reconnecting)
- Remote management via Tailscale was completely lost

### Root cause
The OPNsense installer VM, sitting on the production bridge, almost certainly caused either an IP conflict on the 192.168.1.0/24 subnet or began responding to DHCP/ARP in a way that disrupted the existing gateway. Symptoms observed via the UniFi controller (visible from a separate device on the production network) showed repeated device offline/online cycles consistent with a Layer 2/3 conflict.

### Why remote recovery wasn't possible
Once the Proxmox host lost its Tailscale connection, no remote commands could reach it. This is a hard limit — an offline host cannot be reached. Recovery required physical access to the host.

### Recovery
On returning to the host:
1. Stopped VM 102 via the Proxmox web UI (network flapping ceased within seconds)
2. Verified host reconnected to Tailscale
3. Rebuilt the network architecture with proper isolation (the implementation documented above)

### Lessons learned
- **Never place an untrusted or experimental VM's WAN on the production management bridge.** "The LAN is isolated" is not enough if the WAN can interfere with the host's own connectivity.
- **Plan the blast radius before powering on.** Network changes that touch production should be configured for failure containment first.
- **Layered defense matters.** The final architecture uses three independent layers — VLAN segmentation, network isolation rules in UniFi, and DHCP guarding — so that no single misconfiguration in the lab can affect production.
- **Out-of-band access has limits.** Tailscale is great for remote management but cannot recover a host that has been knocked off the network entirely.

---

## Why This Architecture Is Safe

The redesigned network prevents recurrence of the outage through three independent controls:

1. **VLAN isolation (Layer 2):** Lab traffic is tagged as VLAN 66 and physically segregated from production traffic on the trunk.
2. **Network isolation rules (Layer 3):** The UniFi controller's "Isolate Network" setting blocks routing between VLAN 66 and any other network — only internet egress is permitted.
3. **DHCP guarding:** Even if OPNsense (or any other VM) attempts to serve DHCP on its WAN side, the UniFi switch drops those packets because only 192.168.2.1 is a trusted DHCP source on VLAN 66.

If OPNsense is misconfigured, compromised, or replaced with a malicious VM, the blast radius is contained to the Lab VLAN. Production devices cannot be reached, and production DHCP/gateway services cannot be disrupted.

---

