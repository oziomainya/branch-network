# PeakRetail Ltd: Branch Network Deployment

> **Category:** Networking
> 
> **Status:** Live
> 
> **Course:** Cisco Networking Devices and Initial Configuration · Junior Cybersecurity Analyst Career Path
> 
> **Author:** Ozioma Inya         <!-- REPLACE with your name -->
> 
> **Date:** August, 2025              <!-- REPLACE e.g. "March, 2025" -->

---

## Overview

A complete network infrastructure built and deployed for a new PeakRetail Ltd branch office from scratch. The branch network uses two VLANs (Sales and Management), inter-VLAN routing on the branch router, per-VLAN DHCP pools, a WAN link to headquarters, SSH for secure remote management, and full ICMP verification.

---

## Client Brief

| Detail | Info |
|---|---|
| Client | PeakRetail Ltd |
| Industry | Retail |
| Site | New branch office: 25 staff across Sales and Management |
| Task | Deploy, configure, and verify branch network before opening day |

**Deployment Context:** HQ has designed the network on paper. You are the on-site
network engineer responsible for bringing it online. The branch must connect to HQ
via WAN, segment staff into two VLANs, and be remotely manageable via SSH from HQ.

---

## Objectives

- [x] Document the hierarchical network design and justify device placement
- [x] Explain how cloud and virtualisation could extend the branch network
- [x] Apply binary and hex to design the IPv4 addressing scheme
- [x] Configure VLANs on the branch switch and verify MAC address table
- [x] Configure inter-VLAN routing using router-on-a-stick
- [x] Observe and document the ARP process across VLANs
- [x] Configure per-VLAN DHCP pools and verify automatic address assignment
- [x] Analyse TCP and UDP behaviour in the branch context
- [x] Perform full IOS CLI configuration of both devices including SSH
- [x] Verify end-to-end connectivity using ping, traceroute, and show commands

---

## Tools & Technologies

| Tool | Purpose |
|---|---|
| Cisco Packet Tracer | Network simulation: VLANs, routing, DHCP, SSH |
| Cisco IOS CLI | Full router and switch configuration |
| Packet Tracer Simulation Mode | Observing ARP, TCP, and inter-VLAN packet flow |
| ICMP Ping and Traceroute | End-to-end and cross-VLAN connectivity verification |

---

## Skills Demonstrated

`Hierarchical Network Design` `Cloud and Virtualisation` `Binary and Hex`
`VLAN Configuration` `Trunk Links` `Inter-VLAN Routing` `Router-on-a-Stick`
`MAC Address Table` `Network Layer` `IPv4 Packet Structure`
`IPv4 Addressing` `Subnetting` `ARP` `DHCP per VLAN`
`DNS Configuration` `TCP` `UDP` `Port Numbers`
`Cisco IOS CLI` `SSH Configuration` `Device Hardening`
`ICMP` `Ping` `Traceroute` `Network Troubleshooting`

---

## Network Topology

```
  [HQ-R1: 10.0.0.1] ---------- WAN: 10.0.0.0/30
        |
  [Branch-R1: S0/0/1: 10.0.0.2]
  [Branch-R1: G0/0.10: 192.168.1.1]  VLAN 10 SALES
  [Branch-R1: G0/0.20: 192.168.2.1]  VLAN 20 MANAGEMENT
        | (trunk)
  [Branch-SW1: VLAN20 mgmt: 192.168.2.2]
     |           |            |
  [Fa0/1]     [Fa0/2]      [Fa0/3]
  VLAN 10     VLAN 10      VLAN 20
  Sales-PC1   Sales-PC2    Admin-PC
  DHCP .21    DHCP .22     DHCP .21
```

### Addressing Table

| Device | Interface | IP Address | VLAN | Role |
|---|---|---|---|---|
| Branch-R1 | G0/0.10 | 192.168.1.1 | VLAN 10 | Sales gateway |
| Branch-R1 | G0/0.20 | 192.168.2.1 | VLAN 20 | Management gateway |
| Branch-R1 | S0/0/1 | 10.0.0.2 | -- | WAN uplink to HQ |
| Branch-SW1 | VLAN 20 | 192.168.2.2 | VLAN 20 | Switch management IP |
| Sales-PC1 | NIC | DHCP (192.168.1.21) | VLAN 10 | Sales workstation |
| Sales-PC2 | NIC | DHCP (192.168.1.22) | VLAN 10 | Sales workstation |
| Admin-PC | NIC | DHCP (192.168.2.21) | VLAN 20 | Management workstation |
| HQ-R1 | S0/0/0 | 10.0.0.1 | -- | HQ router / WAN peer |

---

## Lab Parts

### Part 01: Hierarchical Network Design

Three-layer hierarchical model applied to the branch:
- **Access layer**: Branch-SW1 connects end devices, enforces VLANs
- **Distribution/Core (collapsed)**: Branch-R1 for inter-VLAN routing, DHCP, WAN

Design decisions: Two VLANs chosen to segment Sales and Management traffic,
reduce broadcast domains, and enable future QoS policy per VLAN. Router-on-a-stick
chosen over Layer 3 switch since Branch-R1 is already present for WAN.

---

### Part 02: Cloud and Virtualisation

| Cloud Model | PeakRetail Application |
|---|---|
| SaaS | Retail management software, email, collaboration |
| PaaS | IT development and testing environments |
| IaaS | Scalable infrastructure for seasonal demand spikes |

Key insight: Cloud adoption increases WAN criticality. Branch operations
depending on cloud services make the WAN link an operational dependency.

---

### Part 03: Binary and Hex Applied to Addressing

```
192.168.1.1 in binary:
11000000.10101000.00000001.00000001

/24 mask (255.255.255.0):
11111111.11111111.11111111.00000000
  Network portion (24 bits)  Host (8 bits = 254 usable)

/30 mask for WAN (255.255.255.252):
11111111.11111111.11111111.11111100
  Only 2 host bits = 2 usable addresses; perfect for point-to-point
```

---

### Part 04: VLAN Configuration

```
-- Create VLANs
Branch-SW1(config)# vlan 10
Branch-SW1(config-vlan)# name SALES
Branch-SW1(config)# vlan 20
Branch-SW1(config-vlan)# name MANAGEMENT

-- Assign ports
Branch-SW1(config)# interface FastEthernet0/1
Branch-SW1(config-if)# switchport mode access
Branch-SW1(config-if)# switchport access vlan 10

-- Configure trunk to router
Branch-SW1(config)# interface GigabitEthernet0/1
Branch-SW1(config-if)# switchport mode trunk

Branch-SW1# show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/4, Fa0/5, Fa0/6, Fa0/7
                                                Fa0/8, Fa0/9, Fa0/10, Fa0/11
                                                Fa0/12, Fa0/13, Fa0/14, Fa0/15
                                                Fa0/16, Fa0/17, Fa0/18, Fa0/19
                                                Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                Fa0/24, Gig0/1, Gig0/2
10   SALES                            active    Fa0/1, Fa0/2
20   MANAGEMENT                       active    Fa0/3
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active
```

---

### Part 05: Inter-VLAN Routing

```
-- Router-on-a-stick subinterfaces
Branch-R1(config)# interface GigabitEthernet0/0.10
Branch-R1(config-subif)# encapsulation dot1Q 10
Branch-R1(config-subif)# ip address 192.168.1.1 255.255.255.0

Branch-R1(config)# interface GigabitEthernet0/0.20
Branch-R1(config-subif)# encapsulation dot1Q 20
Branch-R1(config-subif)# ip address 192.168.2.1 255.255.255.0

Branch-R1(config)# interface GigabitEthernet0/0
Branch-R1(config-if)# no shutdown
```

---

### Part 06: ARP Process

Cross-VLAN ARP process (Sales-PC1 to Admin-PC):
1. Sales-PC1 checks: 192.168.2.21 is not in its subnet
2. ARPs for default gateway: "Who has 192.168.1.1?"
3. Branch-R1 G0/0.10 replies with its MAC
4. Sales-PC1 sends packet to Branch-R1
5. Branch-R1 ARPs on VLAN 20 for Admin-PC
6. Branch-R1 forwards to Admin-PC

---

### Part 07: Per-VLAN DHCP

```
Branch-R1(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.20
Branch-R1(config)# ip dhcp excluded-address 192.168.2.1 192.168.2.20

Branch-R1(config)# ip dhcp pool VLAN10_SALES
Branch-R1(dhcp-config)# network 192.168.1.0 255.255.255.0
Branch-R1(dhcp-config)# default-router 192.168.1.1
Branch-R1(dhcp-config)# dns-server 10.0.0.1
Branch-R1(dhcp-config)# domain-name peakretail.local

Branch-R1(config)# ip dhcp pool VLAN20_MGMT
Branch-R1(dhcp-config)# network 192.168.2.0 255.255.255.0
Branch-R1(dhcp-config)# default-router 192.168.2.1
Branch-R1(dhcp-config)# dns-server 10.0.0.1
```

---

### Part 08: Transport Layer

| Protocol | Type | Port | Used in Branch For |
|---|---|---|---|
| TCP | Connection-oriented | 22 | SSH management sessions |
| TCP | Connection-oriented | 80/443 | HQ retail system access |
| UDP | Connectionless | 53 | DNS resolution |
| UDP | Connectionless | 67/68 | DHCP address assignment |
| UDP | Connectionless | 123 | NTP time synchronisation |

---

### Part 09: Full IOS CLI Configuration with SSH

```
-- SSH setup on Branch-R1
Branch-R1(config)# ip domain-name peakretail.local
Branch-R1(config)# username admin secret SSH!Admin2024
Branch-R1(config)# crypto key generate rsa
How many bits in the modulus [512]: 1024
% Generating 1024 bit RSA keys, keys will be non-exportable...[OK]
Branch-R1(config)# ip ssh version 2
Branch-R1(config)# line vty 0 4
Branch-R1(config-line)# transport input ssh
Branch-R1(config-line)# login local

-- Verify SSH from Admin-PC
Admin-PC> ssh -l admin 192.168.2.1
Password: SSH!Admin2024
Branch-R1#
```

---

### Part 10: Full Verification

```
-- Connectivity matrix
Sales-PC1 -> Sales-PC2 (VLAN 10)       SUCCESS
Sales-PC1 -> Admin-PC  (cross-VLAN)    SUCCESS
Sales-PC1 -> HQ-R1     (WAN)           SUCCESS
Admin-PC  -> Branch-SW1 (management)   SUCCESS

-- Traceroute Sales-PC1 to HQ
Sales-PC1> tracert 10.0.0.1

Tracing route to 10.0.0.1 over a maximum of 30 hops: 

  1   1 ms      0 ms      0 ms      192.168.1.1
  2   3 ms      3 ms      3 ms      10.0.0.1

Trace complete.

-- Key show commands
Branch-R1# show ip route
Branch-R1# show ip dhcp binding
Branch-SW1# show vlan brief
Branch-SW1# show interfaces GigabitEthernet0/1 trunk
Branch-R1# show ip ssh
```

---

## Results

| # | Result | Status |
|---|---|---|
| 1 | VLANs 10 and 20 correctly configured and verified | OK |
| 2 | Inter-VLAN routing working via router-on-a-stick | OK |
| 3 | Per-VLAN DHCP assigned correct addresses to all clients | OK |
| 4 | WAN connectivity confirmed from both VLANs | OK |
| 5 | SSH v2 configured on both devices, verified by Admin-PC session | OK |
| 6 | Inter-VLAN pings failed because G0/0 physical interface in shutdown | WARNING |
| 7 | Fixed with no shutdown on G0/0 | FIXED |
| 8 | SSH key generation failed as domain name not set first | WARNING |
| 9 | Fixed by setting ip domain-name before crypto key command | FIXED |
| 10 | Admin-PC received VLAN 10 address because Fa0/3 defaulted to VLAN 1 | WARNING |
| 11 | Fixed with switchport access vlan 20 on Fa0/3 | FIXED |

---

## Lessons Learned

**1. Router-on-a-stick requires the physical interface to be up**

Subinterfaces only work if the parent physical interface is active.
Always check physical interface state first when subinterfaces fail.

**2. SSH configuration has a strict dependency order**

Hostname > ip domain-name > username > crypto key generate > ip ssh version 2.
Skipping or reordering any step breaks SSH configuration.

**3. VLAN port assignments are not automatic**

Every port defaults to VLAN 1. Always verify with show vlan brief
after assigning ports to ensure no device is in the wrong VLAN.

**4. ARP across VLANs goes through the gateway**

Intra-VLAN: device ARPs directly for destination.
Inter-VLAN: device ARPs for gateway, router handles delivery.

**5. DHCP pools must match VLAN subnets exactly**

The pool network statement must match the subinterface subnet.
Verify immediately with show ip dhcp pool after configuration.

---

## Repository Structure

```
branch-network/
|-- index.html                    <- Full project write-up (live via GitHub Pages)
|-- README.md                     <- This file
|-- branch-network.pkt            <- Cisco Packet Tracer project file
|-- topology.png                  <- Full branch topology
|-- vlan-brief.png                <- show vlan brief output
|-- inter-vlan-ping.png           <- Cross-VLAN ping from Sales-PC1 to Admin-PC
|-- dhcp-binding.png              <- show ip dhcp binding on Branch-R1
|-- routing-table.png             <- show ip route on Branch-R1
|-- ssh-session.png               <- Admin-PC SSH session to Branch-R1
+-- ping-statistics.png           <- Complete ping matrix
```

---

## Links

- [Full Project Write-up](https://oziomainya.github.io/branch-network)
- [Portfolio](https://oziomainya.github.io)

<!-- REPLACE [YOUR_GITHUB_USERNAME] and [YOUR_PORTFOLIO_URL] with your real details -->

---

*Ozioma Inya · [LinkedIn ↗](https://linkedin.com/in/ozioma-inya-a46327304) · [Github ↗](https://github.com/oziomainya)*
<!-- REPLACE the placeholders above with your real details -->
