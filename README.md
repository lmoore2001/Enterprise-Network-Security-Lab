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

## Roadmap
- [x] Phase 1: Proxmox infrastructure, host networking, updates, and verification
- [ ] Phase 2: Virtual machines, virtual bridges, and network segmentation
- [ ] Phase 3: Windows Server, Active Directory, DNS, and DHCP labs
- [ ] Phase 4: Firewall policies, routing, and access controls
- [ ] Phase 5: Security monitoring, incident investigation, and lab write-ups

## Next Steps
Create the first virtual machine, map virtual network interfaces, and design isolated lab networks so experiments do not affect the home LAN. Record network diagrams, test commands, results, and screenshots for each exercise.
