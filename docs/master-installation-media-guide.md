# Master Installation Media Reference — Enterprise Network Security Lab

**Purpose:** Track the official OS installers, driver media, and virtual appliances needed across Phases 1–6. This is a **reference and planning guide**, not proof that a VM or phase is complete. Choose supported versions available when each phase begins and verify official checksums.

**Host:** Dell OptiPlex 9010 / Proxmox VE 9.2.x / 16 GiB RAM. **Network safety:** Keep the lab isolated on `vmbr1`; do not attach the lab DHCP server to the home LAN (`vmbr0`).

## Quick inventory

| Phase | Media | Format | Status / need | Official source |
|---|---|---|---|---|
| 1 — Proxmox | Proxmox VE installer | ISO | **Already downloaded/installed** | https://www.proxmox.com/en/downloads |
| 2 — Ubuntu networking | Ubuntu Server LTS amd64 | ISO | **Already downloaded/installed** for Linux VMs | https://ubuntu.com/download/server |
| 3 — AD, DNS, DHCP | Windows Server 2025 Evaluation **installation** media (Desktop Experience) | ISO | **Already downloaded/installed** on DC01 | https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2025 |
| 3 — Windows guests | Fedora VirtIO Windows guest drivers (`virtio-win.iso`) | ISO | **Already downloaded** (`virtio-win-0.1.302.iso`); **reuse** for Windows client | https://github.com/virtio-win/virtio-win-pkg-scripts/blob/master/README.md |
| 3 — Domain-joined client | Windows 11 Enterprise Evaluation x64 (regular Enterprise, not LTSC unless specifically desired) | ISO | **NEXT DOWNLOAD**; supports domain join | https://www.microsoft.com/en-us/evalcenter/evaluate-windows-11-enterprise |
| 4 — Firewall | OPNsense DVD amd64 installer (`*-dvd-amd64.iso.bz2` or equivalent release package) | Compressed ISO; extract first | **Future download** | https://opnsense.org/get-started/ |
| 5 — SIEM | Ubuntu Server LTS amd64 **or** official Wazuh virtual appliance | ISO **or** OVA | **Reuse Ubuntu ISO** for fresh Wazuh install; alternatively download OVA; **do not need both** | https://ubuntu.com/download/server ; https://documentation.wazuh.com/current/deployment-options/virtual-machine/virtual-machine.html |
| 6 — Pen testing | Kali Linux installer amd64 | ISO | **Future download** (unless a usable Kali VM already exists) | https://www.kali.org/get-kali/ |
| 6 — Vulnerable targets | Optional deliberately vulnerable lab VM images (e.g. Metasploitable) | Usually VM disk/OVA, **not necessarily ISO** | Optional; only if Phase 6 exercises require | https://docs.rapid7.com/metasploit/metasploitable-2/ |

## Phase-by-phase notes

### Phase 1 — Proxmox VE
- Download the Proxmox VE **ISO Installer** from the official download page to create a **physical bootable USB** for the Dell. This is **not** a guest VM ISO.
- **Already completed**; no need to download again unless rebuilding the hypervisor.
- Keep recovery media and record the downloaded version/checksum.

### Phase 2 — Linux and isolated networking
- Ubuntu Server LTS **amd64** installer ISO is sufficient for Linux VM 101/102. **Reuse the same ISO** for additional Ubuntu guests when appropriate.
- The Proxmox virtual bridge `vmbr1` is a configuration, not a separate download.
- No additional ISO required just for networking tests.

### Phase 3 — Windows Server and Windows client
- DC01 (VM 140) uses **Windows Server 2025 Evaluation installation ISO**, not the separate Windows Server *Languages and Optional Features* ISO. The latter caused an earlier nonbootable-media problem.
- Select **Desktop Experience** in Windows Server setup for GUI administration.
- `virtio-win-0.1.302.iso` is **driver media**, not an operating system. Keep it attached as a second CD/DVD for storage and network drivers; install QEMU guest tools after Windows boots. Reuse it for WIN-CLIENT01.
- **Next download:** Windows 11 Enterprise Evaluation ISO for VM 150. Use the regular Enterprise evaluation, not an unrelated Windows Server ISO. Evaluation licensing/expiration rules apply.
- For Windows 11 on Proxmox, configure **OVMF (UEFI)**, EFI disk, **TPM 2.0**, and suitable virtual hardware. Depending on virtual hardware, load VirtIO storage and network drivers during/after setup.
- Windows 11 client must be attached only to `vmbr1`; DHCP/DNS will be provided by DC01 on `192.168.100.20`.
- Official Microsoft Windows Server ISO and VirtIO ISO are **already obtained**; no reason to redownload unless corrupt or missing.

### Phase 4 — OPNsense firewall
- Download the **DVD/ISO amd64** installer, rather than the USB-specific VGA or serial `.img` images.
- OPNsense releases may distribute the ISO compressed as `.iso.bz2`: **extract the `.iso` first**, then upload that ISO to Proxmox.
- Design interfaces carefully: the future WAN/uplink must not accidentally bridge isolated DHCP into the home LAN. Plan routing/firewall rules before attaching additional NICs.

### Phase 5 — Wazuh security monitoring
- **Recommended for learning:** Reuse the Ubuntu Server LTS ISO and install Wazuh components via official packages/scripts. This provides Linux and SIEM administration experience.
- **Alternative:** Wazuh publishes a **prebuilt OVA virtual appliance**, which is **not an ISO**. Import/conversion steps differ from uploading an ISO to Proxmox; validate compatibility and resources first.
- **Do not download both** unless deliberately comparing approaches.
- Wazuh all-in-one can be memory-intensive. The host has only **16 GiB total**; stop unused VMs and review Wazuh's current minimum requirements before deployment.
- Official docs: https://documentation.wazuh.com/current/quickstart.html

### Phase 6 — Penetration testing and vulnerability assessment
- Download the official **Kali Linux Installer amd64 ISO** for a fresh Proxmox VM. Official prebuilt QEMU VM images are an alternative, but **not ISOs**.
- Other tools (Nmap, Wireshark, OpenVAS/Greenbone, Metasploit, etc.) are generally **installed as software/packages**, not separate operating-system ISOs.
- Optional vulnerable practice targets often ship as virtual disks/appliances. Only use targets within your isolated lab and authorized testing scope.
- Verify Kali images using the official published SHA256/checksum and signing guidance.

## Upload and reuse in Proxmox

1. Download on your laptop from the **official vendor website**.
2. Verify SHA256 hash against the vendor's published checksum when provided (Windows PowerShell: `Get-FileHash -Algorithm SHA256 .\filename.iso`).
3. In Proxmox: **pve → local (pve) → ISO Images → Upload**.
4. Attach ISO to the intended VM under **Hardware → CD/DVD Drive**. For Windows guests, attach both Windows installation ISO and VirtIO driver ISO.
5. Check **Options → Boot Order**: OS installation ISO first during install, then the VM disk first after installation. A driver ISO can stay attached without being a boot target.
6. **Do not delete** an ISO still used by a VM; remove old versions only after verifying they are no longer needed.

## Download checklist

- [x] Phase 1: Proxmox VE ISO (completed).
- [x] Phase 2: Ubuntu Server ISO (completed).
- [x] Phase 3: Windows Server 2025 Evaluation ISO (completed).
- [x] Phase 3: VirtIO Windows drivers ISO (completed).
- [ ] **Phase 3: Windows 11 Enterprise Evaluation ISO — download next.**
- [ ] Phase 4: OPNsense DVD/ISO installer — download when starting firewall phase.
- [ ] Phase 5: Choose Ubuntu reuse **or** Wazuh OVA (not both required).
- [ ] Phase 6: Kali Linux installer ISO (or reuse existing Kali VM if suitable).
- [ ] Phase 6: Optional vulnerable target images only if needed.

## Documentation and troubleshooting reminders
- Record **exact filename, version, source URL, SHA256, download date, target VM, and installation result** for each item.
- Avoid Windows Server *Languages and Optional Features* ISO when you need a **bootable OS installer**.
- VirtIO `vioscsi` driver is for VirtIO SCSI controllers; `NetKVM` is for VirtIO NICs.
- A prebuilt OVA, compressed archive, and ISO are different media types; they are **not interchangeable** in Proxmox's ISO uploader.
- Download only what the current phase requires. This minimizes storage pressure and confusion.
