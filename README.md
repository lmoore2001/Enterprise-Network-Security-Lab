# Enterprise Network & Security Home Lab

## Overview
Hands-on enterprise networking and cybersecurity lab for practicing virtualization, IP addressing, Linux server administration, Windows infrastructure, troubleshooting, and security monitoring. This project is being documented with configuration details, evidence, and verification steps.

## Lab Hardware
| Device | Role |
|---|---|
| Dell OptiPlex 9010 — Intel Core i7-3770, 16 GB RAM, 476.9 GiB SSD + 232.9 GiB HDD | Proxmox virtualization host |
| Raspberry Pi 4 | Planned Linux services and security testing |
| Raspberry Pi Zero W | Planned Wi-Fi monitoring |
| Unmanaged NETGEAR switch | Physical Ethernet connectivity |
| Windows laptop | Web-based administration |

## Phase 1 — Proxmox Host Setup (Complete)
**Detailed checklist, troubleshooting, and screenshots:** [Issue #1 — Install and Configure Proxmox VE](https://github.com/lmoore2001/Enterprise-Network-Security-Lab/issues/1)

### Implementation
1. Audited the Dell's CPU, memory, storage, and Intel VT-x support; confirmed the existing Kali installation did not need to be retained.
2. Prepared a Proxmox VE installer USB using Rufus; selected the 476.94 GiB SATA SSD as the installation target, leaving the separate HDD unselected.
3. Installed Proxmox VE 9.2 (web UI initially displayed 9.2.2), using ext4, and verified the node and local storage appeared in the dashboard.
4. Connected the host by Ethernet through the NETGEAR switch to the Xfinity gateway.
5. Set management hostname `pve.homelab.internal`, static address `10.0.0.234/24`, gateway `10.0.0.1`, and DNS `75.75.75.75`.
6. Created a router-side reserved-IP entry mapping `10.0.0.234` to Dell MAC `B8:CA:3A:AD:39:44` to prevent DHCP from offering the address to another client. Verified the gateway lists the reservation.
7. Disabled Proxmox/Ceph enterprise package repositories (no subscription), enabled the Proxmox `pve-no-subscription` trixie repository, and successfully refreshed package metadata.
8. Completed the initial package upgrade, rebooted for the new kernel, and verified the management dashboard remained accessible at `https://10.0.0.234:8006`.

### Verification
- Web interface reachable from a separate Windows laptop on the same `10.0.0.0/24` LAN.
- Repository refresh finished with `TASK OK`.
- Upgrade completed with `Your System is up-to-date`.
- Host rebooted and the web UI loaded afterward.
- Evidence screenshots attached to [Issue #1](https://github.com/lmoore2001/Enterprise-Network-Security-Lab/issues/1).

### Troubleshooting and Lessons Learned
- **PXE boot error (`PXE-E53`):** The Dell attempted a network boot; used F12 to select the intended UEFI boot entry. Root boot-order cause was not conclusively determined.
- **USB capacity mismatch:** Memorex USB physically labeled 16 GB reported ~28.9 GiB in Windows. The initial H2testw run only tested a 6 MB partition; this did **not** establish the drive's full capacity.
- **Enterprise update error:** The default enterprise package source required a subscription. Switched to the supported no-subscription repository for a home lab.
- **Static IP vs DHCP:** A static host IP inside the gateway's DHCP pool can conflict with future leases; resolved with a router-side reservation for the same address.

## Phase 2 — Ubuntu Server & Virtual Networking (Implementation Complete; Evidence Pending)
**Tracking and detailed verification:** [Issue #2](https://github.com/lmoore2001/Enterprise-Network-Security-Lab/issues/2)

### Implemented
- **VM 100 `ubuntu-server-01`**: Ubuntu Server 24.04.5 LTS; 2 vCPU, 2 GiB RAM, 25 GiB disk; VirtIO NIC on home-LAN bridge `vmbr0`. Last observed DHCP IP `10.0.0.160/24`; gateway `10.0.0.1`. LAN SSH, internet, DNS, and guest agent verified.
- **VM 101 `ubuntu-lab-01`**: Ubuntu clone attached to isolated `vmbr1`, static `192.168.100.10/24`, no configured default gateway.
- **VM 102 `ubuntu-lab-02`**: Ubuntu clone attached to isolated `vmbr1`, static `192.168.100.11/24`, no default gateway (terminal output reviewed).
- Operator reported cleaning cloned hostnames, machine IDs, SSH host keys, and copied Tailscale service state on VM 101/102. Independent comparison output is not yet archived.
- **Tailscale on VM 100**: `100.92.220.12`. Operator successfully SSH'd into VM 100 from a phone on cellular data, confirming offsite remote access. This does not imply direct remote access to the Proxmox dashboard.

### Network Topology
```text
Home LAN 10.0.0.0/24 — router 10.0.0.1
           |
      nic0 / vmbr0 — Proxmox pve 10.0.0.234
           |
      VM 100 Ubuntu — 10.0.0.160 (DHCP)
           +-- Tailscale 100.92.220.12 <--- cellular phone (SSH verified)

      vmbr1 — isolated virtual-only lab network 192.168.100.0/24
           |                                   |
      VM 101 Ubuntu                       VM 102 Ubuntu
      192.168.100.10                      192.168.100.11
           <--------- ping succeeded ---------->
      VM 102 -> 10.0.0.1 ping failed as intended
```

### Verification and Limitations
- Successful VM 101 ↔ VM 102 pings support lab network connectivity.
- VM 102 showed only the connected `192.168.100.0/24` route; home gateway `10.0.0.1` ping failed. These checks support initial isolation but are not exhaustive proof of security boundaries.
- Screenshots of the Proxmox bridge/NIC configuration, guest route and ping tests, and remote SSH remain to be added. Confirm `vmbr1` has no physical uplink and both lab VMs have no extra NICs before declaring all boundary checks complete.
- Keep Proxmox management on `vmbr0`; do not expose test systems to the home LAN or public internet. Do not publish credentials, SSH private keys, or Tailscale auth tokens.

## VM Inventory — Verified vs Planned
| System | Status | Purpose |
|---|---|---|
| Proxmox `pve` | Verified | Dell virtualization host, 16 GiB RAM |
| VM 100 Ubuntu Server | Verified | Linux, SSH, guest agent, Tailscale |
| VM 101 / 102 Ubuntu lab | Implemented; operator tests passed | Isolated network tests on `vmbr1` |
| Windows Server + Windows client | Planned | Phase 3 AD DS, DNS, DHCP, GPO |
| OPNsense VM | Planned | Phase 4 routing and firewall rules |
| Wazuh VM | Planned | Phase 5 SOC and log monitoring |
| Kali + vulnerable training target | Planned | Phase 6 authorized security testing |

With only 16 GiB RAM, not all planned VMs should run simultaneously. VM IDs and IP addresses in future guides are proposals until deployed.

## Reference Guides
These are **runbooks, not completion evidence**. Detailed instructions and career-focused **Why it matters** explanations appear in each issue's first comment.

| Phase | Reference |
|---|---|
| 2 — Virtual networking and remote access | [Issue #3](https://github.com/lmoore2001/Enterprise-Network-Security-Lab/issues/3) |
| 3 — Windows Server, Active Directory, DNS, DHCP | [Issue #5](https://github.com/lmoore2001/Enterprise-Network-Security-Lab/issues/5) |
| 4 — OPNsense firewall and routing | [Issue #6](https://github.com/lmoore2001/Enterprise-Network-Security-Lab/issues/6) |
| 5 — Wazuh monitoring and incident investigation | [Issue #7](https://github.com/lmoore2001/Enterprise-Network-Security-Lab/issues/7) |
| 6 — Penetration testing and vulnerability assessment | [Issue #8](https://github.com/lmoore2001/Enterprise-Network-Security-Lab/issues/8) |

**Optional side projects:** Raspberry Pi 4 Pi-hole DNS filtering and Uptime Kuma availability monitoring. Neither is required for Phase 3.

## Roadmap
- [x] Phase 1: Proxmox infrastructure, host networking, updates, and verification
- [x] Phase 2: Virtual machines, virtual bridges, and network segmentation (core implementation and tests; evidence capture pending)
- [ ] Phase 3: Windows Server, Active Directory, DNS, and DHCP labs
- [ ] Phase 4: Firewall policies, routing, and access controls
- [ ] Phase 5: Security monitoring, incident investigation, and lab write-ups
- [ ] Phase 6: Authorized penetration testing, vulnerability assessment, remediation, and retesting

## Next Steps
Finish Phase 2 evidence: capture redacted screenshots, confirm both VM NICs and the `vmbr1` host bridge configuration, review tailnet security and router port-forwarding settings. Then begin Phase 3 Windows Server and Active Directory.
