# Project Protocols Reference Table
## All Protocols Used in the CCNA Enterprise Lab Project

| Protocol | Layer (OSI) | Application in This Project |
|---|---|---|
| **802.1Q (Dot1Q)** | Layer 2 | VLAN tagging on trunk links between switches and routers (ROAS subinterfaces and core-to-access trunks) |
| **STP / RSTP (802.1w)** | Layer 2 | Loop prevention across the HQ switching fabric; root bridge election tuned via priority on SW-HQ-CORE1 |
| **LACP (802.3ad)** | Layer 2 | EtherChannel bundling of physical links between HQ core and access switches for bandwidth aggregation and redundancy |
| **Port Security** | Layer 2 | Restricts MAC addresses per access port on HQ and Branch access switches; sticky MAC + violation modes |
| **DHCP Snooping** | Layer 2 | Builds a trusted IP-MAC-port binding table on access switches; blocks rogue DHCP servers on untrusted ports |
| **Dynamic ARP Inspection (DAI)** | Layer 2 | Validates ARP packets against the DHCP snooping binding table to prevent ARP spoofing / MITM attacks |
| **HSRP** | Layer 3 | Active/Standby default gateway redundancy for HQ VLANs between SW-HQ-CORE1 (active) and SW-HQ-CORE2 (standby) |
| **IPv4** | Layer 3 | Primary addressing scheme across all sites, point-to-point links, VLANs, loopbacks, and NAT translations |
| **IPv6** | Layer 3 | Dual-stack addressing on all major segments; used with OSPFv3, EIGRPv6, SLAAC, DHCPv6, and IPv6 ACLs |
| **OSPFv2** | Layer 3 | Multi-area IPv4 routing within HQ (Area 0 backbone + Area 1 for access VLANs); ABR role on SW-HQ-CORE1 |
| **OSPFv3** | Layer 3 | IPv6 equivalent of OSPFv2; runs per-interface within HQ using link-local addresses for adjacency |
| **EIGRP (IPv4)** | Layer 3 | Interior routing within the Branch network (AS 100) between R-BR-EDGE and SW-BR-CORE |
| **EIGRPv6** | Layer 3 | IPv6 routing within the Branch network under the same Named EIGRP process (address-family ipv6) |
| **Route Redistribution** | Layer 3 | Mutual OSPF ↔ EIGRP redistribution at the HQ/Branch boundary (R-HQ-EDGE and R-BR-EDGE) with route tagging to prevent loops |
| **Static Routing** | Layer 3 | Default routes toward the ISP (`0.0.0.0/0`) on both edge routers; summary routes back to each site on R-ISP |
| **Floating Static Route** | Layer 3 | Backup default route with elevated AD (200) over the private WAN link, activates only if the primary ISP route fails |
| **NAT / PAT (Dynamic Overload)** | Layer 3 | Translates all internal HQ and Branch RFC-1918 addresses to the public interface IP for Internet access |
| **Static NAT** | Layer 3 | Fixed one-to-one mapping of WEB-SRV (10.10.30.10) to a public IP (203.0.113.10) for inbound access |
| **DHCP (IPv4)** | Layer 3/7 | Router-based DHCP pools on R-HQ-EDGE and R-BR-EDGE serving all user, voice, and management VLANs; relayed via `ip helper-address` on core SVIs |
| **IP Helper (UDP Relay)** | Layer 3 | Relays DHCP broadcast traffic across routed SVI boundaries from clients to the remote DHCP server |
| **SLAAC** | Layer 3 | Stateless IPv6 autoconfiguration for VLAN 10 (Users) hosts using Router Advertisements with M-bit off |
| **DHCPv6 (Stateful)** | Layer 3/7 | Stateful IPv6 address assignment for VLAN 20 (Voice) where predictable addressing is required; M-bit and O-bit set in RAs |
| **Standard ACL** | Layer 3 | Filters SSH/Telnet management access to core switches — permits only the MGMT VLAN (10.10.99.0/24) on VTY lines |
| **Extended ACL (IPv4)** | Layer 3/4 | Granular traffic control on the Server VLAN SVI — permits HTTP/HTTPS from Users, blocks Voice VLAN entirely |
| **Named IPv6 ACL** | Layer 3/4 | IPv6 equivalent of the extended ACL protecting the Server VLAN; explicitly permits ICMPv6 ND messages to avoid breaking neighbor discovery |
| **SSH (v2)** | Layer 7 | Encrypted remote management of all network devices; Telnet explicitly disabled on all VTY lines |
| **NTP** | Layer 7 | Time synchronization across all devices; R-ISP acts as NTP stratum 3 master, all others sync to it for consistent log timestamps |
| **Syslog** | Layer 7 | Centralized event logging; all devices send informational-level logs to the management server (10.10.99.50) with millisecond timestamps |

---

> **Total: 28 protocols and features** across Layers 2 through 7.
>
> **Note:** The Layer 2 security trio (Port Security → DHCP Snooping → DAI) is a deliberate dependency chain — DAI is completely non-functional without the binding table that DHCP Snooping creates first.
