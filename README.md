# NETWORKWALKS-B083-WEEK-1--MAKARANGA-PRAISE-CYBERSECURITY-LAB-SETUP
Cybersecurity internship with network walks lab setup ,installation of virtual box and kali linux .

# 🔐 Cybersecurity Lab Environment Setup — Week 1
## 📌 Project Overview

This project documents the setup of a personal **virtual cybersecurity lab** using Oracle VirtualBox and Kali Linux 2026.2. The goal was to build an isolated environment where security tools, network reconnaissance, and other hands-on cybersecurity exercises can be practiced safely without touching a real production network.

The lab runs on a dedicated NAT Network so that additional virtual machines (targets, other attacker boxes, etc.) can be added later and all communicate with each other while still reaching the internet.

## 🎯 Objectives

- Install VirtualBox as the hypervisor.
- Resolve hardware virtualization (VT-x) issues preventing the VM from starting.
- Import Kali Linux 2026.2 as a virtual machine.
- Create a private **NAT Network** for the lab.
- Configure the Kali VM's network adapter and verify connectivity.
- Take a clean VM snapshot as a recovery baseline.
- Document the process, including problems encountered and how they were solved.

## 🏗️ Lab Architecture

The lab is built around a single custom NAT Network (`NatNetwork`) inside VirtualBox, with the Kali Linux VM attached to it. This keeps the VM isolated from the host's main network while still allowing internet access through NAT and leaving room to add more VMs to the same network later.

## ⚙️ Lab Configuration

| 🧩 Component        | ⚙️ Configuration                              |
| ------------------- | ---------------------------------------------- |
| 🖥️ Host machine     | Lenovo ThinkPad (personal laptop)              |
| 🧰 Hypervisor        | Oracle VirtualBox                              |
| 🐉 Guest OS          | Kali Linux 2026.2                              |
| 🌐 Virtual Network    | NAT Network — `NatNetwork`                     |
| 📡 IPv4 Prefix        | `10.0.0.0/24`                                  |
| 📡 IPv6 Prefix        | `fd17:625c:f037::/64` (IPv6 disabled)          |
| 🚦 DHCP              | Enabled                                        |
| 🔌 Kali Network Adapter | Wired connection 1 (Ethernet, via NAT Network) |

# 🪜 Lab Setup Procedure

## Step 1. Install VirtualBox

VirtualBox was installed on the host ThinkPad as the hypervisor to run the Kali Linux VM.

## Step 2. Resolve the VT-x Virtualization Error

On first attempting to start the Kali VM, VirtualBox failed to boot the machine with the following error:

```
Not in a hypervisor partition (HVP=0) (VERR_NEM_NOT_AVAILABLE).
VT-x is disabled in the BIOS for all CPU modes (VERR_VMX_MSR_ALL_VMX_DISABLED).
Result Code: E_FAIL (0x80004005)
Component: ConsoleWrap
Interface: IConsole
```

This meant hardware virtualization was disabled at the firmware level. It was resolved by:

1. Rebooting into the ThinkPad BIOS (Security tab).
2. Enabling **Intel (R) Virtualization Technology**.
3. Saving and exiting BIOS.
4. Disabling Windows Hyper-V (`bcdedit /set hypervisorlaunchtype off`) and turning off Core Isolation / Memory Integrity, since these can silently block VirtualBox from accessing VT-x even once it's enabled in the BIOS.
5. Restarting the machine and starting the Kali VM again — it powered on successfully afterward.

## Step 3. Create the NAT Network

A dedicated NAT Network was created in VirtualBox under **Network → NAT Networks**:

```
Network Name: NatNetwork
IPv4 Prefix:  10.0.0.0/24
IPv6 Prefix:  fd17:625c:f037::/64
DHCP:         Enabled
IPv6:         Disabled
```

A NAT Network was chosen (rather than a plain NAT adapter) because it allows multiple VMs attached to it to talk to each other directly while each still gets outbound internet access — important for building a lab that can later include target machines alongside Kali.

## Step 4. Import and Configure Kali Linux

The Kali Linux 2026.2 VM was imported into VirtualBox and its network adapter attached to the `NatNetwork` created above. Inside the guest, the network connection appears as **Wired connection 1** under the Ethernet Network menu, and DHCP assigned it an address from the `10.0.0.0/24` range automatically.

## Step 5. Take a Clean Snapshot

Once the VM was confirmed working, a snapshot named **"My first snapshot"** was taken from the VirtualBox Snapshots view. This snapshot represents the clean baseline of the lab — if a future exercise breaks or misconfigures the VM, it can be restored back to this known-good state.

---

# 🔎 Lab Verification

| ✅ Test                     | 🧾 Command / Action                          | 🎯 Expected Result                     |
| --------------------------- | --------------------------------------------- | --------------------------------------- |
| 🌐 Check VM boots            | Start the VM in VirtualBox                    | Kali desktop loads without VT-x error   |
| 📡 Check network connection  | Ethernet Network menu → Wired connection 1    | Connected, DHCP address assigned        |
| 🌍 Browse the internet       | Open Firefox ESR from the Kali desktop        | Kali homepage loads successfully        |
| 🔄 Verify snapshot           | Snapshots tab → confirm "My first snapshot"   | Snapshot listed with correct timestamp  |
| 📁 Host↔Guest file access    | VirtualBox File Manager                       | Guest session opens after login         |

---

# 🐞 Problems Encountered & Solutions

## Problem 1. VirtualBox VT-x / Virtualization Error

**Symptom:** VM failed to start with `VERR_NEM_NOT_AVAILABLE` and "VT-x is disabled in the BIOS for all CPU modes."

**Cause:** Hardware virtualization (Intel VT-x) was disabled in the ThinkPad's BIOS/UEFI firmware, and Windows Hyper-V was also competing for access to it.

**Fix:** Enabled Intel Virtualization Technology in BIOS (Security tab), disabled Hyper-V and Core Isolation/Memory Integrity in Windows, then restarted. The VM started normally afterward.

## Problem 2. Guest File Manager Session Not Open

**Symptom:** The VirtualBox File Manager showed the host file system but the guest side required logging in with a username and password before files could be transferred between host and guest.

**Fix:** Guest Additions and a valid session login are required for the integrated File Manager to browse the guest file system — this is expected behavior and just needs credentials entered in the "Open Session" prompt.

---

# 💡 What I Learned

### 1. VT-x and BIOS-Level Virtualization
Hardware virtualization must be explicitly enabled in the BIOS/UEFI before any hypervisor (VirtualBox, VMware, Hyper-V) can run 64-bit guest VMs — and on Windows hosts, Hyper-V itself can block access to VT-x even after it's enabled in firmware.

### 2. NAT Network vs. Plain NAT
A NAT Network is different from a simple NAT adapter: it allows multiple VMs on the same virtual network to talk to each other directly, while still translating traffic out to the internet — essential for a lab that will eventually include multiple machines.

### 3. Snapshots as a Safety Net
Taking a clean snapshot immediately after getting the VM into a working state provides a reliable rollback point before running riskier exercises later.

### 4. Documenting Problems is Part of the Process
Recording the exact error messages, their causes, and the fixes applied (rather than just the "happy path") makes the lab far more useful to revisit and easier to troubleshoot again in the future.

# 🔐 Security & Ethical Use

This lab is intended strictly for personal learning and authorized testing on systems I own. It will not be used against any system without explicit permission.

# 🔗 Tools & Resources

- **VirtualBox:** https://virtualbox.org/wiki/Downloads
- **Kali Linux:** https://kali.org/get-kali
## 📌 Project Information

**Project:** Cybersecurity & Pentesting Lab Setup | **Week:** 01
