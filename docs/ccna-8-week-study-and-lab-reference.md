# CCNA 200-301 — Eight-Week Study & Home Lab Reference Guide

> **Purpose:** A reusable study guide mapped to the existing Enterprise Network & Security Home Lab phases. This is a **learning plan, not evidence of completed work**. Eight weeks is a foundation pass, not a guaranteed exam-ready timeline. Consult Cisco's official exam topics for the version in effect on your exam date.

## How to use this guide
1. Study the **concepts**; explain them without notes.
2. Do the **Packet Tracer** activity for Cisco IOS switching/routing topics.
3. Connect the concept to the relevant **Proxmox phase** and capture real verification evidence.
4. Record **expected result → observed result → explanation → troubleshooting**.
5. Review the **revisit list** at the end of each week; do not mark lab work complete until verified.
6. Aim for ~60% labs/troubleshooting, 25% learning/notes, 15% subnetting and questions; adjust to weaknesses.

### Current environment (as documented; confirm before relying on it)
- **Phase 1:** Proxmox host management `10.0.0.234/24` on home `10.0.0.0/24`, gateway `10.0.0.1`; management bridge `vmbr0`.
- **Phase 2:** Ubuntu VM 100 on `vmbr0`, last observed `10.0.0.160/24`; isolated `vmbr1` with Ubuntu VM 101 `192.168.100.10/24` and VM 102 `192.168.100.11/24`, no default gateways; ping between them reported successful. Evidence and fuller boundary checks remain pending. Tailscale SSH to VM 100 from cellular was reported successful.
- **Remote dashboard:** [Issue #9](../issues/9) is a **reference guide**, not completion evidence; independently verify current dashboard access and method before claiming completion.
- **Phase 3:** Windows Server VM 140/DC01 `192.168.100.20/24` on `vmbr1`, AD/DNS `corp.lab.test`, DHCP role installed/authorized; DHCP scope, client lease, domain-joined client, and GPO remain to do according to [Issue #11](../issues/11).
- **Phase 4–6:** OPNsense routing/firewall, Wazuh monitoring, and authorized testing are planned.
- **Hardware:** 16 GiB Proxmox host RAM; don't run every VM simultaneously. The unmanaged NETGEAR switch **cannot be used to practice VLAN trunks or STP**; use Packet Tracer (or a separately designed virtual switching environment).

## Exam domain map
| Domain | Weight | Weeks | Primary lab phases |
|---|---:|---|---|
| Network Fundamentals | 20% | 1, ongoing | 1, 2, 3 |
| Network Access | 20% | 2–3 | 2; Packet Tracer |
| IP Connectivity | 25% | 4–5 | 2, 4; Packet Tracer |
| IP Services | 10% | 6 | 1, 3, 4 |
| Security Fundamentals | 15% | 7 | 3, 4, 5, 6 |
| Automation & Programmability | 10% | 8 | All phases |

*Use the official Cisco blueprint as the source of truth for detailed objectives and future version changes.*

---

## Week 1 — Network fundamentals and subnetting
**Study:** OSI vs TCP/IP; frames vs packets; Ethernet, MAC/IP, ARP, TCP/UDP, ports; IPv4 masks/CIDR, VLSM, IPv6 basics; gateways, DNS and DHCP; cabling/interface characteristics.

**Map to phases:** Revisit **Phase 1** host `10.0.0.234/24` and gateway; **Phase 2** `vmbr0` versus isolated `vmbr1`; **Phase 3** DC01 static `192.168.100.20/24`.

**Practice**
- Calculate network, first/last usable, broadcast, and host count for `10.0.0.234/24`, `192.168.100.10/24`, `192.168.100.64/27`, and `192.168.100.100/26`.
- Remember: conventional IPv4 usable hosts = `2^(32-prefix) - 2` (special cases `/31` and `/32`). `192.168.100.64/27` has 32 total, 30 usable (`.65–.94`), broadcast `.95`.
- From Ubuntu VMs: `ip -br addr`, `ip route`, `ip neigh`, `resolvectl status`, `ping`, `tracepath` if installed. On Windows: `ipconfig /all`, `route print`, `arp -a`, `nslookup`.
- Predict what VM 101 can reach without a default route; explain *why* a failed ping alone does not prove complete isolation.

**Evidence:** Addressing table, subnet calculations, redacted route/interface output, annotated `vmbr0`/`vmbr1` diagram.

**Revisit before moving on:** Can you explain MAC vs IP, broadcast domains, subnet masks, ARP, and why a host needs a default gateway for off-subnet traffic?

## Week 2 — Switching, VLANs and trunks
**Study:** MAC learning/forwarding/flooding; collision vs broadcast domains; VLAN membership; access vs trunk ports; 802.1Q tagging, native VLAN; inter-VLAN routing basics.

**Map to phases:** **Phase 2** isolation is conceptually related to segmentation, but `vmbr1` is a **separate Linux bridge, not proof you configured an 802.1Q VLAN**. Keep the distinction explicit. **Phase 4** will introduce deliberate inter-network routing.

**Packet Tracer:** Build VLAN 10 Users, VLAN 20 Servers, VLAN 99 Management; configure access ports, a trunk, and router-on-a-stick or an L3 switch. Test same-VLAN and inter-VLAN traffic. Use `show vlan brief`, `show interfaces trunk`, `show mac address-table`.

**Evidence:** Topology, switch configs, verification outputs, before/after ping matrix.

**Revisit:** Why does a VLAN require Layer 3 routing to communicate with another VLAN? What happens if a trunk does not permit the VLAN?

## Week 3 — STP, EtherChannel and wireless
**Study:** STP/RSTP purpose, root bridge, port roles/states, PortFast/BPDU Guard concepts; LACP/EtherChannel; WLAN SSIDs, bands/channels, AP/controller concepts, WPA2/WPA3.

**Map to phases:** The existing unmanaged NETGEAR switch does not expose Cisco STP/LACP/VLAN configuration. Practice in **Packet Tracer**; later correlate wireless concepts with home Wi-Fi without changing production settings.

**Packet Tracer:** Three-switch triangle; identify root bridge and blocked alternate path; remove a link and observe convergence. Configure LACP between compatible switches. Draw a simple AP/client topology.

**Verify:** `show spanning-tree`, `show etherchannel summary`, `show interfaces status`.

**Revisit:** Why do redundant Layer 2 paths create loops? What is LACP negotiating? How do interference and channel overlap affect Wi-Fi?

## Week 4 — Routing tables, static routes, IPv6
**Study:** Connected/local/static/default routes; longest-prefix match; administrative distance vs metric; next hop; forwarding decisions; IPv6 addressing and routing.

**Map to phases:** Compare **Phase 2** VM 100's default route via `10.0.0.1` with VM 101/102's connected-only route. Prepare for **Phase 4** OPNsense but do not add a gateway to isolated VMs until the firewall design is intentional.

**Packet Tracer:** Three routers with separate LANs; configure static/default routes, then deliberately remove a return route and troubleshoot. Practice `show ip route`, `show ipv6 route`, `show ip interface brief`, `ping`, `traceroute`.

**Evidence:** Routing tables, forward and return path explanation, failure-and-fix record.

**Revisit:** Can a host send off-subnet traffic without a default route? Why must the destination have a return path? How is the longest matching prefix chosen?

## Week 5 — OSPF and troubleshooting
**Study:** Single-area OSPFv2; neighbor relationships, router IDs, advertised networks, areas, costs; route selection and systematic fault isolation.

**Map to phases:** **Phase 4** routing concepts. OPNsense is useful for routing/firewall policy, but **Packet Tracer Cisco routers** are the main OSPF IOS practice environment.

**Packet Tracer:** Three-router area 0 topology; establish neighbors; verify learned routes; break an interface/network statement and troubleshoot. Commands: `show ip ospf neighbor`, `show ip protocols`, `show ip route ospf`, `show running-config`.

**Evidence:** Diagram, adjacency states, route table, intentionally broken config and correction.

**Revisit:** Why would OSPF neighbors fail to form? How does OSPF differ from static routing? Can you locate a fault using Layer 1 → Layer 3 checks?

## Week 6 — IP services: DHCP, DNS, NAT, NTP, SSH
**Study:** DHCP DORA, relay concept, options; DNS resolution and record types; NAT vs PAT; NTP, syslog, SNMP; SSH management and QoS basics.

**Map to phases:**
- **Phase 1:** Static Proxmox IP and router reservation; understand reservation vs host static configuration.
- **Phase 3 (next hands-on priority):** On DC01, create DHCP scope `192.168.100.100–192.168.100.150`, option 006 DNS `192.168.100.20`, option 015 domain `corp.lab.test`; **omit option 003 gateway** until Phase 4. Keep DHCP bound only to `vmbr1`. Attach an isolated test client, verify lease, DNS, and domain resolution.
- **Phase 4:** Implement and document intentional NAT/PAT and firewall policy on OPNsense; do not inadvertently route test traffic onto the home LAN.

**Verify:** Windows `Get-DhcpServerv4Scope`, `Get-DhcpServerv4OptionValue`, `ipconfig /all`, `ipconfig /renew`, `Resolve-DnsName corp.lab.test`; Linux `resolvectl status` and `ip route`. Record the DHCP discover/offer/request/ack sequence via Packet Tracer simulation or Wireshark where feasible.

**Revisit:** DNS is not DHCP; a DNS server does not act as a gateway. Why can an isolated client resolve local domain records without internet access?

## Week 7 — Security fundamentals and ACLs
**Study:** Standard vs extended ACLs, implicit deny, ACL placement; switch port security, DHCP snooping, Dynamic ARP Inspection; AAA, RADIUS/TACACS+, SSH hardening, WLAN security, least privilege and VPN concepts.

**Map to phases:**
- **Phase 3:** Create fictional AD users/groups, complete domain join and test a GPO; distinguish identity/group policy from packet-filtering ACLs.
- **Phase 4:** Create explicit OPNsense allow/deny rules between lab segments, then test both positive and negative paths. Preserve home LAN isolation.
- **Phase 5:** Log firewall events to monitoring when implemented.
- **Phase 6:** Only test authorized isolated targets.

**Packet Tracer:** Extended ACL: allow a client subnet to reach a web service, deny SSH to the same server, verify with traffic and counters. Practice `show access-lists` and interface ACL checks.

**Revisit:** What is the implicit deny? Why does rule order matter? What is the difference between an ACL, firewall state tracking, and an AD permission?

## Week 8 — Automation, review and exam-style troubleshooting
**Study:** Controller-based vs traditional networking, northbound/southbound APIs, REST/HTTP methods, JSON, configuration management concepts, network automation and AI/ML operations at blueprint level.

**Map to phases:** Review all phases. Use the existing **GitHub repository** as a documentation/automation portfolio. Build a small, safe Python script that parses **sample JSON** device/interface data and prints a summary; no production credentials.

**Practice:** Rebuild a small Packet Tracer network from scratch: VLANs, trunk, inter-VLAN routing, DHCP, OSPF and an ACL. Troubleshoot three deliberate failures. Take a timed practice assessment and sort misses by domain.

**Evidence:** Sanitized JSON example, script, screenshots of successful/failed verification, knowledge-gap list.

**Revisit:** Can you read JSON, explain GET vs POST vs PUT vs DELETE, and describe what controllers change about network operations?

---

## Phase-to-CCNA crosswalk: what to revisit while progressing

| Home lab phase | CCNA topics to revisit | Concrete next action | Evidence/status |
|---|---|---|---|
| **1 — Proxmox host** | IPv4/CIDR, default gateway, DNS, Ethernet, DHCP reservations, basic troubleshooting | Explain `10.0.0.234/24` vs `10.0.0.1`; diagram management traffic | Host setup documented; revisit as study |
| **2 — Linux/isolated networking** | Layer 2 bridging, IP routes, ARP, subnetting, isolation, SSH | Capture `vmbr1` configuration, VM NIC membership, route tables and boundary tests; distinguish bridges from VLANs | Core work done; evidence pending |
| **3 — Windows Server/AD** | DHCP, DNS, IPv4, client addressing, authentication vs network security | Finish isolated DHCP scope, client lease, domain join, GPO; capture DORA and DNS tests | DC/AD/DNS deployed; DHCP scope/client/GPO pending |
| **4 — OPNsense** | Routing, default/static routes, NAT/PAT, ACL/firewall, troubleshooting | Design routed subnets and least-privilege rules **before** enabling forwarding | Planned |
| **5 — Wazuh** | Syslog, SNMP concepts, security monitoring, incident workflows | Ingest lab logs and correlate denied connections | Planned |
| **6 — Security testing** | ACL validation, segmentation, least privilege, network troubleshooting | Test only authorized isolated targets; document detection and remediation | Planned |

### Priority catch-up list from existing GitHub tracking
- [ ] **Phase 2:** Save redacted bridge/NIC screenshots, `ip route` outputs, connectivity tests, and independent isolation checks ([Issue #2](../issues/2)).
- [ ] **Remote access:** Confirm actual Proxmox-dashboard access method, offsite verification, and tailnet rules; don't treat [reference Issue #9](../issues/9) as proof.
- [ ] **Phase 3:** Configure isolated DHCP scope/options; capture lease from a client, then domain join and GPO ([Issue #11](../issues/11)).
- [ ] **Packet Tracer:** Build VLAN/trunk, STP/LACP, OSPF, and Cisco ACL labs—these aren't demonstrated by Proxmox bridges alone.
- [ ] **Phase 4:** Draw route/NAT/firewall plan and test in isolation before changing traffic paths.

## Reusable lab documentation template
```md
### Topic / CCNA domain
### Related home lab phase and issue
### Objective and predicted outcome
### Topology / addressing
### Commands and configuration (redacted)
### Verification: expected vs actual
### Troubleshooting: symptom → hypothesis → test → fix
### Why it matters (CCNA + real IT/security work)
### Screenshots/evidence links
### What I still need to review
```

## Essential command quick reference
| Platform | Useful commands |
|---|---|
| Cisco IOS | `show ip interface brief`, `show vlan brief`, `show interfaces trunk`, `show mac address-table`, `show spanning-tree`, `show etherchannel summary`, `show ip route`, `show ip ospf neighbor`, `show access-lists` |
| Linux | `ip -br addr`, `ip route`, `ip neigh`, `resolvectl status`, `ss -tulpen`, `ping`, `tracepath` |
| Windows | `ipconfig /all`, `route print`, `arp -a`, `nslookup`, `Resolve-DnsName`, `Get-NetIPConfiguration`, `Get-DhcpServerv4Scope` |

## References
- [Cisco CCNA certification and official exam topics](https://www.cisco.com/site/us/en/learn/training-certifications/certifications/enterprise/ccna/index.html)
- [Cisco Packet Tracer / Networking Academy](https://www.netacad.com/)
- [Phase 1](../issues/1) · [Phase 2](../issues/2) · [Phase 3](../issues/11) · [Phase 4 reference](../issues/6) · [Phase 5 reference](../issues/7) · [Phase 6 reference](../issues/8)
