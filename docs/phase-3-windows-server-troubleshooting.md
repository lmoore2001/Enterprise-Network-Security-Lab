# Phase 3 — Windows Server 2025 deployment and troubleshooting

**Date:** October 2026  
**Lab:** Proxmox VE 9.2.x, Dell OptiPlex 9010 (16 GiB host RAM)  
**VM:** 140 / DC01 — 1 socket, 2 vCPU, 4 GiB RAM, 60 GiB SCSI disk on local-lvm; isolated bridge vmbr1; VirtIO NIC; VirtIO SCSI single controller.

## Verified progress
- Created VM 140 with Windows Server 2025 Evaluation installation ISO and virtio-win-0.1.302.iso on separate virtual CD/DVD drives.
- Confirmed bootable Windows Setup, reached disk selection, subsequently reached Windows Server lock/sign-in screen, and continued into Server Manager.
- Began Add Roles and Features Wizard using **Role-based or feature-based installation** and local server from the **server pool**; user reported AD DS installation in progress. **Completion is not yet verified.**
- Windows network adapter configuration, static IPv4, guest agent, AD DS promotion, DNS and DHCP operation require verification.

## Troubleshooting chronology

### 1. No bootable device / CD-ROM boot error
**Symptom:** SeaBIOS reported `Boot failed: not a bootable disk`, `Could not read from CDROM (code 0004)`, then attempted PXE and looped.

**Investigation:** Original attached Microsoft ISO filename contained `SERVER_LOF_PACKAGES_OEM`, indicating language/optional-features media rather than a normal bootable Windows Server installer. Verified Proxmox CD/DVD attachments and boot order.

**Action/outcome:** Attached Windows Server 2025 Evaluation installation ISO (filename containing `SERVER_EVAL_x64FRE_en-us.iso`) on ide2 and VirtIO ISO on ide0. Put ide2 ahead of scsi0; disabled network and VirtIO ISO as boot targets while leaving VirtIO media attached. Windows Setup subsequently booted successfully.

### 2. Windows logo stall and VM configuration lock
**Symptom:** Windows startup sat on the logo; Proxmox CPU graph around 50% of two assigned vCPUs. Changing processor configuration produced `can't lock file '/var/lock/qemu-server/lock-140.conf' - got timeout (500)`.

**Investigation/action:** Proxmox showed VM 140 still running and processor type `host` as a pending change from `x86-64-v2-AES`. Recommended stopping VM cleanly before retrying startup and applying pending CPU configuration. Later Windows Setup was reached.

**Limitation:** The specific cause of the stall and whether CPU-type change alone fixed it were **not conclusively verified**. Do not present the CPU type as a proven fix. Avoid manually deleting lock files without checking active VM tasks.

### 3. Windows Setup did not detect 60 GiB disk
**Symptom:** Disk selection page was empty even though Proxmox Hardware showed `scsi0` on `local-lvm` (60 GiB) with VirtIO SCSI single controller.

**Investigation/action:** Attached Fedora VirtIO driver ISO; browsed to `D:\\vioscsi\\2k25\\amd64`. Enabled **Hide drivers that aren't compatible with this computer's hardware**; Windows displayed a matching **Red Hat VirtIO SCSI pass-through controller** from `vioscsi.inf`. Selected matching storage driver for installation.

**Clarification:** Driver display name containing 'pass-through' is not equivalent to enabling PCI passthrough in Proxmox. Earlier troubleshooting also explored `viostor.inf`, but the VM uses VirtIO SCSI, so `vioscsi.inf` is the relevant storage driver.

**Outcome:** Windows later reached lock/sign-in screen and Server Manager, demonstrating successful OS installation.

### 4. Ctrl+Alt+Delete captured by local laptop
**Symptom:** Keyboard shortcut affected the computer running the browser rather than Windows Server guest.

**Action:** Use Proxmox noVNC console toolbar **Send key → Ctrl-Alt-Del**, or try Ctrl+Alt+End with console focus.

### 5. Ethernet adapter not visible in Windows
**Symptom:** No Ethernet adapter found during initial Windows configuration.

**Likely cause/action:** VirtIO network driver not installed yet. Inspect Device Manager → Other devices → Ethernet Controller, then load driver from `NetKVM\\2k25\\amd64` on VirtIO ISO. Verify using `Get-NetAdapter` and `ncpa.cpl`.

**Status:** Driver installation and resulting adapter detection have not been independently confirmed. Keep VM NIC attached **only** to `vmbr1` to avoid exposing future DHCP service to the home LAN.

### 6. Installing AD DS roles
**Question:** Choose installation to offline VHD or server pool? Internet required?

**Action:** Use **Role-based or feature-based installation → Select a server from the server pool → DC01 → Active Directory Domain Services**. AD DS, DNS and domain creation can be performed without internet on the isolated bridge.

**Status:** User reported role installation underway. **Do not mark AD DS complete or promote domain controller yet.** Verify static IPv4 `192.168.100.20/24`, no gateway on isolated network, and intended DNS setup first.

## Next verification checklist
- [ ] Confirm Server Manager reports AD DS role installation successful.
- [ ] Confirm VM CPU type, disk, RAM, and network settings; document screenshot.
- [ ] Confirm VirtIO NetKVM network driver and QEMU Guest Agent installed; verify Proxmox guest agent setting.
- [ ] Rename Windows host DC01 if necessary, set static `192.168.100.20/24` on vmbr1, no gateway; verify with `ipconfig /all`.
- [ ] Promote to new forest `corp.lab.test` and install/configure DNS; verify `dcdiag` and DNS resolution.
- [ ] Configure DHCP scope only on isolated lab network after domain health is confirmed.
- [ ] Record screenshots and actual test output without passwords, private identifiers, or secrets.
