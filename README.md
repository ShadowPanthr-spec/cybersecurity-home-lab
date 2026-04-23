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

---

## Project Overview

This project documents the build-out of a personal cybersecurity home lab using repurposed hardware. The lab is designed to provide hands-on experience in Linux administration, network monitoring, and security tooling to complement existing industry certifications.

The project is structured in three phases:
1. Linux installation and configuration on legacy hardware
2. 2. Network monitoring and server setup
   3. 3. Active security practice and skill building
     
      4. ---
     
      5. ## Phase 1 — Linux Installation on Dell XPS M140 ✅ COMPLETED
     
      6. ### Objective
      7. Install a secure, actively maintained Linux distribution on the Dell XPS M140 to repurpose it as a lab machine.
     
      8. ### OS Selection Process
     
      9. Selecting the right operating system required careful consideration of the hardware's age and architecture limitations. The Dell XPS M140 uses a 32-bit processor, which eliminated many modern distributions.
     
      10. **Option 1 — Debian:**
      11. Debian was the first choice due to its reputation as one of the most secure and stable Linux distributions with excellent long-term support. However, Debian dropped official 32-bit (i386) support after Debian 10 (Buster), making newer versions incompatible with the Dell XPS M140's hardware. Eliminated due to architecture incompatibility.
     
      12. **Option 2 — Devuan:**
      13. Devuan is a Debian fork that removes systemd and was considered as an alternative with potentially better support for older hardware. However, it also presented compatibility issues with the Dell XPS M140's specific hardware configuration. Eliminated due to hardware compatibility issues.
     
      14. **Option 3 — antiX Linux (Selected):**
      15. antiX Linux was chosen as the final option for the following reasons:
      16. - Built specifically for older 32-bit hardware
          - - Actively maintained with ongoing security updates and patches
            - - Based on Debian, inheriting much of its security foundation
              - - Lightweight enough to run efficiently on aging hardware
               
                - > **Key Lesson:** A theoretically more secure OS is irrelevant if it doesn't support your hardware or is no longer receiving updates. Compatibility and active maintenance are critical selection criteria.
                  >
                  > ---
                  >
                  > ### Tools Used
                  >
                  > | Tool | Purpose | Platform |
                  > |---|---|---|
                  > | Rufus | Create bootable USB drive from ISO | LG Gram (Windows 11) |
                  > | antiX Linux 5.10.240-antix.1-486-smp | Linux operating system | Dell XPS M140 |
                  > | Windows PowerShell | Hash verification of downloaded ISO | LG Gram (Windows 11) |
                  >
                  > ---
                  >
                  > ### Step-by-Step Process
                  >
                  > **Step 1 — Download antiX Linux ISO on LG Gram**
                  > Downloaded the antiX Linux ISO file from the official antiX website to the LG Gram primary laptop.
                  >
                  > **Step 2 — Verify ISO Integrity via Hash Check**
                  > Before proceeding with installation, the integrity of the downloaded ISO was verified using Windows PowerShell to ensure the file had not been corrupted or tampered with during download. This is a critical security practice.
                  >
                  > Command used in PowerShell:
                  > ```powershell
                  > Get-FileHash C:\path\to\antix.iso -Algorithm SHA256
                  > ```
                  > The output hash was compared against the SHA256 hash published on the official antiX download page. They matched, confirming the file was legitimate and unmodified.
                  >
                  > **Step 3 — Create Bootable USB Drive**
                  > Used Rufus on the LG Gram to write the verified antiX Linux ISO to a 16GB USB flash drive, creating a bootable installation media.
                  >
                  > **Step 4 — Boot Dell XPS M140 from USB**
                  > Inserted the bootable USB into the Dell XPS M140 and configured it to boot from the USB drive. Successfully booted into the antiX Linux live environment.
                  >
                  > **Step 5 — Install antiX Linux to Internal Hard Drive**
                  > Ran the antiX installer from the live environment, installing the OS to the internal hard drive. During setup, created a regular user account and set a root password. The installer confirmed the OS was installed to:
                  > - Device: `/dev/sda`
                  > - - Partition: `/dev/sda1`
                  >   - - Size: 73.3GB
                  >    
                  >     - **Step 6 — Verify Installation**
                  >     - Before rebooting, opened a terminal from the live USB environment and verified the installation using the `lsblk` command, which confirmed antiX was correctly installed and mounted to the appropriate partition.
                  >    
                  >     - **Step 7 — Remove USB and Reboot**
                  >     - Removed the USB flash drive as instructed and attempted to reboot into the newly installed antiX system.
                  >    
                  >     - ---
                  >
                  > ### Problems Encountered & Solutions
                  >
                  > **Problem 1 — System dropped to `grub>` prompt on reboot**
                  >
                  > | | |
                  > |---|---|
                  > | **Symptom** | After removing the USB and rebooting, the system dropped to a `grub>` command prompt instead of booting into antiX |
                  > | **GRUB Version** | GNU GRUB version 2.12-9+deb13u1 |
                  > | **Cause** | GRUB bootloader loaded successfully but could not locate the Linux kernel automatically |
                  > | **Diagnosis** | Used `ls` command at the `grub>` prompt to confirm GRUB could see the drive: output showed `(hd0) (hd0,msdos1)`. Used `ls (hd0,msdos1)/` to confirm the antiX filesystem was present |
                  > | **Solution** | Manually booted the system using the following GRUB commands |
                  >
                  > Commands used to manually boot:
                  > ```bash
                  > set root=(hd0,msdos1)
                  > linux (hd0,msdos1)/boot/vmlinuz-5.10.240-antix.1-486-smp root=/dev/sda1 ro
                  > initrd (hd0,msdos1)/boot/initrd.img-5.10.240-antix.1-486-smp
                  > boot
                  > ```
                  >
                  > **Problem 2 — Kernel filename not found at default path**
                  >
                  > | | |
                  > |---|---|
                  > | **Symptom** | Initial boot command using `/boot/vmlinuz` returned a file not found error |
                  > | **Cause** | The kernel filename included the full version string rather than a generic name |
                  > | **Solution** | Used `ls (hd0,msdos1)/boot/` to list exact filenames. Identified correct filenames: `vmlinuz-5.10.240-antix.1-486-smp` and `initrd.img-5.10.240-antix.1-486-smp` |
                  >
                  > **Problem 3 — Root login forbidden at console**
                  >
                  > | | |
                  > |---|---|
                  > | **Symptom** | Attempting to log in as `root` returned "root account login forbidden" |
                  > | **Cause** | antiX Linux disables direct root login at the console by default as a security measure |
                  > | **Solution** | Logged in using the regular user account created during installation, then used `sudo` for commands requiring elevated privileges |
                  >
                  > ---
                  >
                  > ### Permanent GRUB Fix
                  >
                  > After successfully booting into antiX, the GRUB configuration was permanently fixed to prevent needing manual boot commands in the future.
                  >
                  > Opened a terminal from the antiX desktop and ran:
                  > ```bash
                  > sudo update-grub
                  > ```
                  > GRUB scanned for operating systems, generated a new configuration file, and added a boot menu entry. On the next reboot, the system booted automatically into antiX without any manual intervention.
                  >
                  > **Outcome:** antiX Linux fully installed, verified, and booting correctly and independently. ✅
                  >
                  > ---
                  >
                  > ### Key Lessons Learned from Phase 1
                  >
                  > - Always document as you go. A session interruption without documentation means starting from scratch and losing hours of work.
                  > - - Hash verification of downloaded files is a simple but essential security practice that should never be skipped.
                  >   - - GRUB issues on older hardware are common but fixable directly from the `grub>` prompt.
                  >     - - Linux root login is typically disabled by default — this is a security feature, not a bug.
                  >       - - Architecture compatibility (32-bit vs 64-bit) is a critical factor when selecting software for older hardware.
                  >         - - antiX Linux is an excellent, actively maintained choice for legacy 32-bit machines.
                  >          
                  >           - ---
                  >
                  > ## Phase 2 — Lab Configuration 🔄 IN PROGRESS
                  >
                  > ### Objective
                  > Configure the Dell XPS M140 running antiX Linux as a network monitoring tool and/or server within the home lab environment.
                  >
                  > **Planned steps:**
                  > - [ ] Connect Dell to home network
                  > - [ ] - [ ] Assess network monitoring tools compatible with antiX (e.g., Wireshark, ntopng, Snort, Suricata)
                  > - [ ] - [ ] Install and configure chosen monitoring tools
                  > - [ ] - [ ] Define and document the Dell's specific role in the lab
                  >
                  > - [ ] ---
                  >
                  > - [ ] ## Phase 3 — Security Practice & Skill Building 📋 PLANNED
                  >
                  > - [ ] ### Objective
                  > - [ ] Use the home lab environment for active, hands-on cybersecurity practice to build skills beyond certification knowledge.
                  >
                  > - [ ] **Planned steps:**
                  > - [ ] - [ ] Set up Hack the Box on LG Gram via browser
                  > - [ ] - [ ] Use Dell for network traffic analysis during practice sessions
                  > - [ ] - [ ] Document all exercises, findings, and lessons learned
                  > - [ ] - [ ] Build toward more advanced certifications and practical experience
                  >
                  > - [ ] ---
                  >
                  > - [ ] ## Resources & References
                  >
                  > - [ ] - antiX Linux: https://antixlinux.com
                  > - [ ] - Rufus: https://rufus.ie
                  > - [ ] - Hack the Box: https://hackthebox.com
                  > - [ ] - CompTIA: https://comptia.org
                  > - [ ] - AWS Certification: https://aws.amazon.com/certification
                  >
                  > - [ ] ---
                  >
                  > - [ ] ## About This Project
                  >
                  > - [ ] This home lab is being built and documented as part of an active journey into the cybersecurity field. All work is conducted in a personal, isolated lab environment for educational purposes.
                  >
                  > - [ ] ---
                  >
                  > - [ ] *Documentation last updated: April 2026*
