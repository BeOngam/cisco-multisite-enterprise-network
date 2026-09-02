# 🌐 CCNA Enterprise Network Lab
### A full-scale, multi-site Cisco network built from scratch in Packet Tracer — covering nearly every topic on the CCNA 200-301 blueprint in one cohesive project.

---

## What is this?

This is a hands-on **CCNA Routing & Switching lab project** designed as a complete refresher course. Instead of practicing each protocol in isolation, everything is wired together into a single enterprise topology — the way it actually works in the real world.

The network spans four logical sites:

- **HQ** — dual-core L3 switching, multi-area OSPF, HSRP redundancy, and a hardened access layer
- **Branch Office** — EIGRP routing, L3 switching, and a simpler (but still production-realistic) design
- **Simulated ISP** — static routing, NTP master, and a loopback to simulate Internet destinations
- **Server Farm** — lives inside HQ's VLAN 30, reachable from the Internet via Static NAT

---

## Topology Overview

```
                              ┌─────────────┐
                              │    R-ISP     │
                              │  8.8.8.8/32  │
                              └──┬───────┬───┘
                   203.0.113.0/30│       │203.0.113.4/30
                    ┌────────────┴─┐   ┌─┴────────────┐
                    │  R-HQ-EDGE   │   │  R-BR-EDGE   │
                    │  1.1.1.1/32  ├───┤  2.2.2.2/32  │
                    │  OSPF + NAT  │   │  EIGRP + NAT │
                    └──────┬───────┘   └──────┬────────┘
                           │                  │
               ┌───────────┴──────┐    ┌──────┴──────────┐
               │  SW-HQ-CORE1/2   │    │   SW-BR-CORE     │
               │  HSRP + OSPF ABR │    │   EIGRP + SVIs   │
               └───┬──────────┬───┘    └──────┬───────────┘
               ┌───┴───┐  ┌───┴───┐       ┌───┴───┐
               │ ACC1  │  │ ACC2  │       │ ACC1  │
               └───────┘  └───────┘       └───────┘
              VLAN 10/20/30/99          VLAN 10/20/99
```

---

## Technologies Covered

### Layer 2
| Feature | Details |
|---|---|
| VLANs | Users (10), Voice (20), Servers (30), Mgmt (99), Blackhole (999) |
| 802.1Q Trunking | Native VLAN hardened, allowed VLANs explicitly set |
| EtherChannel | LACP (802.3ad) between core and access switches |
| RSTP | Priority tuned so CORE1 is always root bridge |
| Port Security | Sticky MAC, max 2 per port, restrict violation mode |
| DHCP Snooping | Trusted uplinks only; rate-limiting on untrusted ports |
| DAI | ARP validation against the DHCP snooping binding table |

### Layer 3
| Feature | Details |
|---|---|
| OSPFv2 / OSPFv3 | Multi-area (Area 0 + Area 1), ABR on CORE1, IPv4 + IPv6 |
| EIGRP / EIGRPv6 | Named mode, AS 100, Branch-only |
| Route Redistribution | Mutual OSPF ↔ EIGRP with route tagging to prevent loops |
| HSRP | Active/Standby per VLAN, MD5 auth, object tracking |
| NAT / PAT | Dynamic overload for internet access + static NAT for web server |
| Static & Floating Routes | Default routes to ISP, floating backup over private WAN (AD 200) |
| IPv4 + IPv6 Dual-Stack | All major segments addressed in both families |
| SLAAC | Stateless autoconfiguration for user VLANs (M-bit off) |
| DHCPv6 Stateful | Predictable IPv6 addressing for voice VLANs (M-bit + O-bit on) |

### Security & Management
| Feature | Details |
|---|---|
| Standard ACL | Restricts VTY access to MGMT VLAN only |
| Extended ACL | Protects server VLAN — HTTP/HTTPS from users only, Voice VLAN blocked |
| Named IPv6 ACL | Same server protection in IPv6, with explicit ICMPv6 ND permits |
| SSH v2 | Only remote access method; Telnet disabled on all VTY lines |
| NTP | R-ISP as stratum 3 master; all devices synchronized |
| Syslog | Informational-level logs sent to 10.10.99.50 with ms timestamps |

---

## IP Addressing (Quick Reference)

### Point-to-Point Links
| Link | Subnet | Side A | Side B |
|---|---|---|---|
| R-HQ-EDGE ↔ R-ISP | 203.0.113.0/30 | .1 | .2 |
| R-BR-EDGE ↔ R-ISP | 203.0.113.4/30 | .1 | .2 |
| R-HQ-EDGE ↔ R-BR-EDGE | 172.16.0.0/30 | .1 | .2 |
| R-HQ-EDGE ↔ SW-HQ-CORE1 | 10.10.253.0/30 | .1 | .2 |
| R-BR-EDGE ↔ SW-BR-CORE | 10.20.253.0/30 | .1 | .2 |

### HQ VLANs (HSRP VIP / CORE1 / CORE2)
| VLAN | Subnet | VIP | CORE1 | CORE2 |
|---|---|---|---|---|
| 10 – Users | 10.10.10.0/24 | .1 | .2 | .3 |
| 20 – Voice | 10.10.20.0/24 | .1 | .2 | .3 |
| 30 – Servers | 10.10.30.0/24 | .1 | .2 | .3 |
| 99 – Mgmt | 10.10.99.0/24 | .1 | .2 | .3 |

### Branch VLANs
| VLAN | Subnet | Gateway (SVI) |
|---|---|---|
| 10 – Users | 10.20.10.0/24 | .1 |
| 20 – Voice | 10.20.20.0/24 | .1 |
| 99 – Mgmt | 10.20.99.0/24 | .1 |

---

## How to Use This

1. **Open Packet Tracer** (8.x recommended) and build the topology following `Section 1` of the lab guide
2. **Add the NM-1FGE module** to R-HQ-EDGE and R-BR-EDGE — both need a third Gigabit interface
3. **Follow the phases in order** — each phase depends on the previous one:
   - Skipping VLANs before SVIs, or DAI before DHCP Snooping, will break things in non-obvious ways
4. **Run the verification commands** after each phase before moving on
5. **Use the final test matrix** in Phase 16 to validate the full end-to-end topology

> If something breaks mid-lab, check the troubleshooting tips at the end of each phase — the most common mistakes for each topic are documented there.

---

## Platform Notes

- **Packet Tracer 8.x** — fully supported. Some Layer 2 security features (DAI, DHCP Snooping) behave differently depending on switch model; use **3560-24PS** for core and **2960-24TT** for access.
- **GNS3 / EVE-NG** — all commands work as-is with IOSv / IOSvL2 images. No substitutions needed.
- The project uses only `2001:DB8::/32` for IPv6 (documentation prefix) — safe for lab use, will never accidentally route on a real network.

---


## Credentials (Lab Use Only)

| Account | Value |
|---|---|
| Enable Secret | `Cisco123!` |
| Username | `admin` |
| Password | `Admin123!` |
| SSH Version | 2 |
| VTY Access | MGMT VLAN only (10.x.99.0/24) |

> Obviously don't use these in production. You already knew that.

---

## License

This project is shared for **educational purposes**. Feel free to use it, modify it, and build on it for your own CCNA studies or teaching material. A credit back to this repo is appreciated but not required.

---

*Built as a full CCNA 200-301 refresher project. If you spot an error or have a suggestion, open an issue — feedback is always welcome.*
