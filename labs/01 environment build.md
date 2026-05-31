# Lab 01 — Environment Build and Troubleshooting

**Date:** March 2026  
**Category:** Infrastructure  
**Difficulty:** Beginner  

---

## Objective

Deploy a virtualized enterprise lab environment consisting of four VMs connected on an isolated internal network. The goal was to mirror the architecture of a real small enterprise network where all machines communicate over a defined subnet with static IP assignments.

---

## Why This Matters

Before you can defend a network or attack one you need an environment to practice in. Building this lab from scratch mirrors exactly what an infrastructure engineer does when a company needs a new environment — design the network, assign addresses, deploy the machines, and make them talk to each other.

---

## Environment

| Component | Details |
|---|---|
| Host Machine | Windows 10, 24GB RAM, AMD processor |
| Hypervisor | VirtualBox 7.2.6 |
| Network | NAT Network, 192.168.10.0/24 |
| VMs | Windows Server 2022, Windows 10, Ubuntu Server, Kali Linux 2025.4 |

---

## Network Design

| VM | Hostname | IP | Role |
|---|---|---|---|
| Windows Server 2022 | ADDC01 | 192.168.10.7 | Domain Controller |
| Windows 10 | TARGET | 192.168.10.100 (DHCP) | Employee endpoint |
| Ubuntu Server | Splunk | 192.168.10.10 | SIEM |
| Kali Linux | Kali | 192.168.10.250 | Attacker |

Static IPs were assigned to all machines except the Windows 10 target which uses DHCP to simulate how a real employee workstation gets its address from the domain controller.

---

## What Broke

**Problem 1: VMs would not boot**
Every VM launched into a black screen with a no bootable medium error. Checked boot order, moved Optical to top, reattached the ISO, changed paravirtualization from Default to KVM for AMD compatibility. None of these fixed it. The Windows Server 2022 ISO was corrupted from the first download. Re-downloading resolved the issue. Ubuntu had booted fine previously confirming the problem was the specific ISO not VirtualBox or the AMD processor.

Lesson: always verify file size after downloading large ISOs. Windows Server 2022 should be approximately 5.6GB.

**Problem 2: Ubuntu static IP — wrong netplan filename**
Following standard instructions to edit /etc/netplan/00-installer-config.yaml opened a blank file because the file did not exist. Running ls /etc/netplan/ revealed the file was named 50-cloud-init.yaml on this Ubuntu install. Different Ubuntu versions create different default netplan filenames.

Lesson: always check what is actually in the directory before editing.

**Problem 3: YAML indentation error**
sudo netplan apply threw an inconsistent indentation error. YAML requires spaces never tabs and the routes section had mixed indentation. Fixed by rewriting the section using only spacebar indentation.

Lesson: YAML is whitespace-sensitive. This applies everywhere — Kubernetes configs, Docker Compose, Ansible playbooks.

**Problem 4: Domain Controller broke after rename**
Renamed the Windows Server VM after it had already been promoted to Domain Controller. On next login got: the security database on the server does not have a computer account for this workstation trust relationship. Tried DSRM recovery mode, netdom commands, PowerShell Remove-Computer. None worked because AD services cannot run in DSRM and network drivers do not load in Safe Mode. Rebuilt the Domain Controller from scratch. The correct order is rename the machine first then promote to Domain Controller.

Lesson: Active Directory records the computer name during domain promotion. Renaming after the fact creates a mismatch between the machine identity and what AD expects. This cannot be fixed without rebuilding.

**Problem 5: VMs could not reach each other**
Some VMs were on NAT and others on NAT Network. These are completely different things in VirtualBox. NAT gives internet access but VMs cannot see each other. NAT Network creates an isolated internal network where VMs can communicate. Fixed by setting every VM to NAT Network with the same network name.

Lesson: VirtualBox has confusingly similar network modes. NAT and NAT Network are not the same thing.

---

## Decisions I Made and Why

**Why static IPs on servers but DHCP on the endpoint**
Servers need consistent addresses so other machines always know where to find them. If the Domain Controller IP changed on reboot DNS would break and no machines could authenticate. The Windows 10 machine uses DHCP because that is how real employee workstations work — the DC hands out addresses automatically when they connect.

**Why NAT Network instead of Bridged**
Bridged mode would expose all VMs to the home network. NAT Network keeps everything isolated so the attack machine can only reach the other lab VMs. This prevents accidentally attacking real devices on the home network and mirrors how an enterprise isolates lab environments.

---

## Resources

- [VirtualBox Network Modes Documentation](https://www.virtualbox.org/manual/ch06.html)
- [Microsoft Windows Server 2022 Evaluation](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022)
- [Ubuntu Netplan Documentation](https://netplan.io/reference)
