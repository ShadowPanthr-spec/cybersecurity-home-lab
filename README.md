# Cybersecurity Home Lab Project

**Project Start Date:** April 2026  
**Certifications:** CompTIA Network+ CE, CompTIA Security+ CE, AWS Cloud Practitioner, AWS Solutions Architect Associate  
**Goal:** Build a hands-on home lab environment to develop practical cybersecurity skills and support career development toward an IT Help Desk role and beyond.

---

## Hardware Inventory

| Device | Model | Specs | Role |
|---|---|---|---|
| Primary Laptop | LG Gram | Intel Core i7-1165G7 @ 2.80GHz, Windows 11 Home, 958GB storage (270GB used) | Main workstation, research, Hack the Box |
| Lab Machine | Dell XPS M140 | 32-bit architecture, 73.3GB HDD | Linux lab server, network monitoring |
| Primary Router | NETGEAR RAX80 | Wi-Fi 6, dual-band, gigabit ports | ISP gateway, living room — hosts home & IoT networks |
| Secondary Router | NETGEAR RS500 | Wi-Fi 6, dual-band, gigabit ports | Office router (double NAT) — hosts lab & bedroom IoT networks |

---

## Project Overview

This project documents the build-out of a personal cybersecurity home lab using repurposed hardware. The lab is designed to provide hands-on experience in Linux administration, network monitoring, and security tooling to complement existing industry certifications.

The project is structured in three phases:

- Linux installation and configuration on legacy hardware
- Network monitoring and server setup
- Active security practice and skill building

---

## Network Architecture

### Design Rationale

The home lab network uses a **double NAT** topology with two NETGEAR routers and multiple VLANs. This design was driven by three requirements:

**Physical Connectivity Constraint:** The living room and office are separated by a bathroom wall that significantly degrades Wi-Fi signal quality. Running a single router to cover both rooms was not viable, so the NETGEAR RS500 in the office is connected to the NETGEAR RAX80 in the living room via a wired WAN uplink. This creates a natural double NAT boundary.

**Security Segmentation:** Cybersecurity lab work requires strict isolation from household devices. Running attack tools, vulnerable VMs, and network scanners on the same broadcast domain as general household devices or smart home kit presents unnecessary risk. The double NAT boundary between routers means lab traffic is fully separated from the household network at the network layer, not just at the VLAN level.

**Household Usability:** The primary household user (mother) and general home devices need a simple, stable network that is completely unaffected by lab activity. Placing the RAX80 as the primary router in the living room ensures all household traffic routes through a clean, unmodified path to the ISP with no exposure to lab VLANs.

---

### Double NAT Topology — Visual Diagram

```
                         INTERNET
                             |
                        [ ISP Modem ]
                             |
                    ┌────────────────┐
                    │  NETGEAR RAX80 │   ← WAN: ISP IP
                    │  (Living Room) │     LAN: 192.168.1.0/24
                    │                │
                    │  VLAN 10 ─────────── Home Network      192.168.1.0/24
                    │  VLAN 20 ─────────── IoT Network       192.168.20.0/24
                    │                │
                    └────────┬───────┘
                             │
                     Wired WAN Uplink
                    (RAX80 LAN port →
                     RS500 WAN port)
                             │
                    ┌────────────────┐
                    │  NETGEAR RS500 │   ← WAN: 192.168.1.x (NAT'd by RAX80)
                    │    (Office)    │     LAN: 10.0.0.0/8
                    │                │
                    │  VLAN 30 ─────────── Cybersecurity Lab  10.0.30.0/24
                    │  VLAN 40 ─────────── Bedroom IoT        10.0.40.0/24
                    │                │
                    └────────────────┘

  ┌──────────────────────────────────────────────────────────┐
  │  KEY: Each arrow (→) represents a NAT boundary.          │
  │  Traffic from the Lab VLAN passes through TWO NAT layers │
  │  before reaching the internet — once at the RS500        │
  │  and once at the RAX80. Lab devices cannot initiate      │
  │  connections to any RAX80-hosted VLAN by design.         │
  └──────────────────────────────────────────────────────────┘
```

---

### Network Segments & VLAN Allocation

| Router | VLAN ID | Network Name | Subnet | Gateway | Purpose |
|---|---|---|---|---|---|
| RAX80 (Living Room) | VLAN 10 | Home Network | 192.168.1.0/24 | 192.168.1.1 | General household use — primary user (mum), shared devices |
| RAX80 (Living Room) | VLAN 20 | Home IoT | 192.168.20.0/24 | 192.168.20.1 | Smart home devices in the living room / common areas |
| RS500 (Office) | VLAN 30 | Cybersecurity Lab | 10.0.30.0/24 | 10.0.30.1 | Lab machines, VMs, attack/defence practice, monitoring tools |
| RS500 (Office) | VLAN 40 | Bedroom IoT | 10.0.40.0/24 | 10.0.40.1 | Bedroom smart devices (Sony TV, etc.) — isolated from lab |

> **Why separate IoT VLANs on each router?**  
> Smart home devices are distributed across rooms. Keeping living room IoT on the RAX80 and bedroom IoT on the RS500 means each router manages only the devices physically nearby, reducing cross-router traffic and keeping IoT devices isolated from both household and lab traffic regardless of which router they connect through.

---

### Device Allocation by Network

#### RAX80 — VLAN 10: Home Network (192.168.1.0/24)

| Device | Type | Assigned IP / Range |
|---|---|---|
| Mother's phone / laptop | Personal devices | DHCP: 192.168.1.50–192.168.1.150 |
| Shared household devices | General use | DHCP: 192.168.1.50–192.168.1.150 |
| NETGEAR RS500 WAN port | Router uplink | Static: 192.168.1.2 |

#### RAX80 — VLAN 20: Home IoT (192.168.20.0/24)

| Device | Type | Assigned IP / Range |
|---|---|---|
| Living room smart devices | IoT (lights, speakers, etc.) | DHCP: 192.168.20.50–192.168.20.150 |

#### RS500 — VLAN 30: Cybersecurity Lab (10.0.30.0/24)

| Device | Type | Assigned IP / Range |
|---|---|---|
| LG Gram (Primary Laptop) | Main workstation | Static: 10.0.30.10 |
| Dell XPS M140 (antiX Linux) | Lab server / monitor | Static: 10.0.30.20 |
| Virtual Machines (future) | Attack / target VMs | DHCP: 10.0.30.100–10.0.30.200 |

#### RS500 — VLAN 40: Bedroom IoT (10.0.40.0/24)

| Device | Type | Assigned IP / Range |
|---|---|---|
| Sony TV | Smart TV | Static: 10.0.40.10 |
| Other bedroom IoT | Smart devices | DHCP: 10.0.40.50–10.0.40.150 |

---

### IP Addressing Summary

| Block | Router | Scope |
|---|---|---|
| 192.168.1.0/24 | RAX80 | Primary home network (VLAN 10) |
| 192.168.20.0/24 | RAX80 | Home IoT network (VLAN 20) |
| 10.0.30.0/24 | RS500 | Cybersecurity lab (VLAN 30) |
| 10.0.40.0/24 | RS500 | Bedroom IoT (VLAN 40) |

The RS500 is deliberately assigned to the `10.0.0.0/8` private address space (vs the RAX80's `192.168.x.x`) to make it visually obvious in logs, packet captures, and monitoring output which router and VLAN a device or packet belongs to. This aids in debugging and keeps mental overhead low during lab sessions.

---

### Security Properties of the Double NAT Design

The double NAT architecture provides the following security properties:

- **Lab-to-home isolation:** Devices on RS500 VLANs (10.0.x.x) cannot initiate connections to RAX80 VLANs (192.168.x.x) unless explicitly port-forwarded — which is not configured. The NAT boundary at the RAX80 blocks all unsolicited inbound connections from the lab.
- **Home-to-lab isolation:** Devices on the RAX80's home network (192.168.1.x) have no route to the RS500's internal subnets (10.0.x.x). The RS500 is simply another client on the home network from the RAX80's perspective.
- **IoT containment:** Both IoT VLANs (VLAN 20 and VLAN 40) are isolated from all other VLANs by VLAN segmentation. Smart devices cannot communicate with lab machines or household computers.
- **Lab traffic double-NAT'd to internet:** Outbound lab traffic is NAT translated at the RS500 (to 192.168.1.x) and again at the RAX80 (to the ISP-assigned public IP), providing an additional layer of address obfuscation.

---

### Known Limitations & Future Improvements

- **No IDS/IPS at the NAT boundary** — a future improvement would be to configure Snort or Suricata on the Dell XPS M140 as an inline sensor on VLAN 30.
- **No inter-VLAN routing logging** — VLAN boundary crossing should be logged. This is planned for Phase 2.
- **Double NAT complicates inbound connectivity** — any services that require inbound port forwarding (e.g., reverse shells in CTF challenges) require double port-forward configuration (on both routers). This is an acceptable trade-off for the isolation benefits.
- **RS500 WAN uplink is on the home network VLAN** — in the current design, the RS500's WAN IP (192.168.1.2) sits on VLAN 10 alongside household devices. A dedicated WAN VLAN on the RAX80 for the uplink would be cleaner but is not currently required.

---

## Phase 1 — Linux Installation on Dell XPS M140 ✅ COMPLETED

### Objective

Install a secure, actively maintained Linux distribution on the Dell XPS M140 to repurpose it as a lab machine.

### OS Selection Process

Selecting the right operating system required careful consideration of the hardware's age and architecture limitations. The Dell XPS M140 uses a 32-bit processor, which eliminated many modern distributions.

**Option 1 — Debian:** Debian was the first choice due to its reputation as one of the most secure and stable Linux distributions with excellent long-term support. However, Debian dropped official 32-bit (i386) support after Debian 10 (Buster), making newer versions incompatible with the Dell XPS M140's hardware. Eliminated due to architecture incompatibility.

**Option 2 — Devuan:** Devuan is a Debian fork that removes systemd and was considered as an alternative with potentially better support for older hardware. However, it also presented compatibility issues with the Dell XPS M140's specific hardware configuration. Eliminated due to hardware compatibility issues.

**Option 3 — antiX Linux (Selected):** antiX Linux was chosen as the final option for the following reasons:

- Built specifically for older 32-bit hardware
- Actively maintained with ongoing security updates and patches
- Based on Debian, inheriting much of its security foundation
- Lightweight enough to run efficiently on aging hardware

> **Key Lesson:** A theoretically more secure OS is irrelevant if it doesn't support your hardware or is no longer receiving updates. Compatibility and active maintenance are critical selection criteria.

### Tools Used

| Tool | Purpose | Platform |
|---|---|---|
| Rufus | Create bootable USB drive from ISO | LG Gram (Windows 11) |
| antiX Linux 5.10.240-antix.1-486-smp | Linux operating system | Dell XPS M140 |
| Windows PowerShell | Hash verification of downloaded ISO | LG Gram (Windows 11) |

### Step-by-Step Process

**Step 1 — Download antiX Linux ISO on LG Gram**  
Downloaded the antiX Linux ISO file from the official antiX website to the LG Gram primary laptop.

**Step 2 — Verify ISO Integrity via Hash Check**  
Before proceeding with installation, the integrity of the downloaded ISO was verified using Windows PowerShell to ensure the file had not been corrupted or tampered with during download. This is a critical security practice.

Command used in PowerShell:
```powershell
Get-FileHash C:\path\to\antix.iso -Algorithm SHA256
```
The output hash was compared against the SHA256 hash published on the official antiX download page. They matched, confirming the file was legitimate and unmodified.

**Step 3 — Create Bootable USB Drive**  
Used Rufus on the LG Gram to write the verified antiX Linux ISO to a 16GB USB flash drive, creating a bootable installation media.

**Step 4 — Boot Dell XPS M140 from USB**  
Inserted the bootable USB into the Dell XPS M140 and configured it to boot from the USB drive. Successfully booted into the antiX Linux live environment.

**Step 5 — Install antiX Linux to Internal Hard Drive**  
Ran the antiX installer from the live environment, installing the OS to the internal hard drive. During setup, created a regular user account and set a root password. The installer confirmed the OS was installed to:

- Device: `/dev/sda`
- Partition: `/dev/sda1`
- Size: 73.3GB

**Step 6 — Verify Installation**  
Before rebooting, opened a terminal from the live USB environment and verified the installation using the `lsblk` command, which confirmed antiX was correctly installed and mounted to the appropriate partition.

**Step 7 — Remove USB and Reboot**  
Removed the USB flash drive as instructed and attempted to reboot into the newly installed antiX system.

### Problems Encountered & Solutions

**Problem 1 — System dropped to `grub>` prompt on reboot**

*Symptom:* After removing the USB and rebooting, the system dropped to a `grub>` command prompt instead of booting into antiX

*GRUB Version:* GNU GRUB version 2.12-9+deb13u1

*Cause:* GRUB bootloader loaded successfully but could not locate the Linux kernel automatically

*Diagnosis:* Used `ls` command at the `grub>` prompt to confirm GRUB could see the drive: output showed `(hd0) (hd0,msdos1)`. Used `ls (hd0,msdos1)/` to confirm the antiX filesystem was present

*Solution:* Manually booted the system using the following GRUB commands

```
set root=(hd0,msdos1)
linux (hd0,msdos1)/boot/vmlinuz-5.10.240-antix.1-486-smp root=/dev/sda1 ro
initrd (hd0,msdos1)/boot/initrd.img-5.10.240-antix.1-486-smp
boot
```

**Problem 2 — Kernel filename not found at default path**

*Symptom:* Initial boot command using `/boot/vmlinuz` returned a file not found error

*Cause:* The kernel filename included the full version string rather than a generic name

*Solution:* Used `ls (hd0,msdos1)/boot/` to list exact filenames. Identified correct filenames: `vmlinuz-5.10.240-antix.1-486-smp` and `initrd.img-5.10.240-antix.1-486-smp`

**Problem 3 — Root login forbidden at console**

*Symptom:* Attempting to log in as root returned "root account login forbidden"

*Cause:* antiX Linux disables direct root login at the console by default as a security measure

*Solution:* Logged in using the regular user account created during installation, then used `sudo` for commands requiring elevated privileges

### Permanent GRUB Fix

After successfully booting into antiX, the GRUB configuration was permanently fixed to prevent needing manual boot commands in the future. Opened a terminal from the antiX desktop and ran:

```bash
sudo update-grub
```

GRUB scanned for operating systems, generated a new configuration file, and added a boot menu entry. On the next reboot, the system booted automatically into antiX without any manual intervention.

**Outcome:** antiX Linux fully installed, verified, and booting correctly and independently. ✅

### Key Lessons Learned from Phase 1

- Always document as you go. A session interruption without documentation means starting from scratch and losing hours of work.
- Hash verification of downloaded files is a simple but essential security practice that should never be skipped.
- GRUB issues on older hardware are common but fixable directly from the `grub>` prompt.
- Linux root login is typically disabled by default — this is a security feature, not a bug.
- Architecture compatibility (32-bit vs 64-bit) is a critical factor when selecting software for older hardware.
- antiX Linux is an excellent, actively maintained choice for legacy 32-bit machines.

---

## Phase 2 — Lab Configuration 🔄 IN PROGRESS

### Objective

Configure the Dell XPS M140 running antiX Linux as a network monitoring tool and/or server within the home lab environment.

Planned steps:
- [ ] Connect Dell to Lab VLAN (RS500 VLAN 30 — 10.0.30.0/24)
- [ ] Assign static IP: 10.0.30.20
- [ ] Assess network monitoring tools compatible with antiX (e.g., Wireshark, ntopng, Snort, Suricata)
- [ ] Install and configure chosen monitoring tools
- [ ] Define and document the Dell's specific role in the lab

---

## Phase 3 — Security Practice & Skill Building 📋 PLANNED

### Objective

Use the home lab environment for active, hands-on cybersecurity practice to build skills beyond certification knowledge.

Planned steps:
- [ ] Set up Hack the Box on LG Gram via browser
- [ ] Use Dell for network traffic analysis during practice sessions
- [ ] Document all exercises, findings, and lessons learned
- [ ] Build toward more advanced certifications and practical experience

---

## Resources & References

- antiX Linux: https://antixlinux.com
- Rufus: https://rufus.ie
- Hack the Box: https://hackthebox.com
- CompTIA: https://comptia.org
- AWS Certification: https://aws.amazon.com/certification
- NETGEAR RAX80: https://www.netgear.com/home/wifi/routers/rax80/
- NETGEAR RS500: https://www.netgear.com/business/wired/routers/

---

## About This Project

This home lab is being built and documented as part of an active journey into the cybersecurity field. All work is conducted in a personal, isolated lab environment for educational purposes.

---

*Documentation last updated: April 2026*
