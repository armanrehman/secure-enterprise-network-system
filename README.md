# Secure Enterprise Network System

Design and implementation of a redundant, segmented enterprise network with a firewalled dual-ISP edge, centralized wireless, and VoIP. Built and verified in Cisco Packet Tracer.

---

## 1. Overview

This project is a complete network design and implementation for a 600-user organization occupying a new three-floor headquarters. The building houses six departments (Sales and Marketing, HR and Logistics, Finance and Accounts, Administration and Public Relations, ICT, and a Server Room), and every department receives wired LAN access, WiFi, and IP telephony. Core services (DHCP, DNS, RADIUS) run in a protected internal server zone, public-facing servers (Web, FTP, Mail, App, NAS storage) run in a DMZ, and the network reaches the internet through two ISPs behind a pair of Cisco ASA firewalls.

The design goal: no single point of failure between an end user and the internet, with security enforced at every layer, and room to grow without redesign.

## 2. Architecture and Key Design Decisions

- **Hierarchical design with a collapsed core.** Two multilayer switches act as a combined core and distribution layer and perform all inter-VLAN routing. Access switches on each floor connect end devices only, which keeps the access layer simple and the design scalable.
- **Redundancy at every tier.** Two ISPs, two ASA firewalls, two core switches. Every access switch uplinks to both cores, and each core connects to both firewalls over routed /30 point-to-point links.
- **Routing without dedicated internal routers.** The firewalls and the multilayer cores carry all routing, reducing device count and keeping policy enforcement at the choke point traffic already crosses.
- **Firewall security zones.** INSIDE (users and internal servers), two OUTSIDE zones (one per ISP), and a DMZ, enforced through ASA security levels, NAT, and inspection ACLs.
- **Server placement by exposure.** Authentication and infrastructure servers (DHCP, DNS, RADIUS) sit inside the firewall in a dedicated server VLAN. Anything reachable from outside (Web, FTP, Mail, App, NAS) sits in the DMZ, so a compromise there cannot reach the internal network directly.
- **Gateway redundancy with load sharing.** HSRP runs on both cores for every VLAN. Hosts point at one virtual gateway per VLAN, and the active role alternates between the cores across VLANs, so both switches forward traffic in normal operation and cover for each other on failure.
- **Addressing strategy.** Clients (wired, wireless, phones) receive addresses from centralized DHCP through relay agents on the gateway SVIs. Every server keeps a static address so firewall rules, NAT, and DNS always point at the right host.
- **Routing strategy.** OSPF advertises all internal networks between cores, firewalls, and edge routers. Each firewall carries a static default route to its primary ISP plus a floating backup route to the second ISP.
- **Layer 2 hardening.** PortFast and BPDU Guard on every access port, all unused ports shut down and parked in an isolated VLAN, and remote management restricted to SSH from the management subnet only.

## 3. Network Segmentation and Addressing

| Segment | VLAN | Subnet | Purpose |
|---|---|---|---|
| Management | 10 | 192.168.10.0/24 | Device management, admin access |
| LAN | 20 | 172.16.0.0/16 | Wired user devices in every department |
| WLAN | 50 | 10.20.0.0/16 | Wireless clients through the WLC |
| VoIP | 70 | 172.30.0.0/16 | IP phones (voice VLAN) |
| Inside Servers | 90 | 10.11.11.32/27 | DHCP, DNS, RADIUS (static addressing) |
| DMZ | zone | 10.11.11.0/27 | Web, FTP, Mail, App, NAS (static addressing) |
| Unused | 199 | none | All spare ports, shut down and isolated |

Core-to-firewall connections use routed /30 point-to-point subnets, and each ISP hands off a public /30 block at the edge.

## 4. Technologies and Skills Implemented (in build order)

1. Base device configuration and hardening: hostnames, console and enable passwords, password encryption, login banners, exec timeouts, disabled DNS lookup on the CLI
2. Secure remote management: SSH v2 with RSA keys, local user authentication, VTY lines locked to SSH and filtered by a standard ACL so only the management subnet can connect
3. VLAN segmentation and 802.1Q trunking across all access and core switches
4. Voice VLAN configuration on phone-facing access ports
5. Unused-port lockdown: every spare port shut down and assigned to an isolated VLAN
6. Spanning Tree PortFast and BPDU Guard on access ports
7. EtherChannel link aggregation with LACP between the core switches
8. Subnetting and IP address planning across all network segments
9. Layer 3 core: routed point-to-point uplinks, SVIs, and inter-VLAN routing on the multilayer switches
10. HSRP first-hop redundancy with per-VLAN active/standby load balancing, priorities, and preemption
11. Centralized DHCP with relay (ip helper-address) on every gateway SVI
12. Static addressing for all DMZ and internal servers
13. OSPF dynamic routing (single area) across core switches, firewalls, and edge routers
14. Cisco ASA firewall setup: interface security levels and zones (inside, dual outside, DMZ)
15. Firewall routing: primary and floating backup static default routes toward the ISPs, plus OSPF for internal reachability
16. NAT/PAT on the ASAs using object NAT (dynamic interface) for LAN, WLAN, and DMZ traffic
17. Firewall inspection policy: extended ACLs applied to the DMZ and both outside interfaces
18. Centralized wireless: Wireless LAN Controller with lightweight access points (CAPWAP) serving every department
19. VoIP: voice gateway running Call Manager Express, dedicated voice VLAN, DHCP Option 150 with TFTP provisioning, automatic phone registration and extension assignment
20. End-to-end verification and failover testing

## 5. What the Finished Network Delivers

- Every department has wired access, WiFi through centrally managed lightweight APs, and a registered IP phone with its own extension.
- All clients receive addressing automatically from central DHCP; all servers sit at fixed, documented addresses.
- LAN, WLAN, and DMZ traffic reaches the internet through NAT on the firewalls, and a working path remains if a core switch, a firewall, or an ISP link fails.
- Only explicitly permitted protocols can enter through the outside and DMZ interfaces; everything else is dropped at the firewall.

## 6. Verification and Testing

Testing covered inter-VLAN reachability from every department VLAN, DHCP lease verification on wired and wireless clients, HSRP failover by disabling the active core, OSPF neighbor and routing table checks, firewall policy tests confirming permitted versus blocked protocols, NAT translation verification, wireless association through the WLC, and phone-to-phone calls across floors.

---

*Design and implementation tool: Cisco Packet Tracer.*
