# Home Lab and Security Portfolio

**Focus:** Enterprise Windows Infrastructure | Threat Detection | Incident Response

**Certs:** CompTIA Security+ | In Progress: AZ-900 and AZ-104

**GitHub:** github.com/creyes-tech

---

## Lab Overview

A virtualized enterprise attack and defense environment built entirely from scratch. Every entry documents what was built, what broke, the decisions made along the way, and what was learned. This is not a tutorial follow-along. It is a working lab that generates real attacks, real telemetry, and real incident response.

**Domain:** tom.local

**Network:** 192.168.10.0/24

| VM | OS | IP | Role |
|---|---|---|---|
| ADDC01 | Windows Server 2022 | 192.168.10.7 | Domain Controller, Active Directory, DNS, DHCP |
| TARGET | Windows 10 | 192.168.10.100 | Simulated employee endpoint |
| Splunk | Ubuntu Server | 192.168.10.10 | SIEM, log collection and threat detection |
| Kali | Kali Linux 2025.4 | 192.168.10.250 | Attack machine |

---

## Labs and Write-ups

| # | Title | Skills | Status |
|---|---|---|---|
| 01 | [Environment Build and Troubleshooting](./labs/01-environment-build.md) | VirtualBox, VM deployment, NAT Network, static IPs, network troubleshooting | Complete |
| 02 | [Active Directory Domain Setup](./labs/02-active-directory.md) | Windows Server 2022, AD DS, domain promotion, DNS, OUs, users, security groups, Group Policy | Complete |
| 03 | [Splunk SIEM Deployment](./labs/03-splunk-siem.md) | Ubuntu Server, Splunk Enterprise, Universal Forwarder, Sysmon, inputs.conf, log ingestion | Complete |
| 04 | [Brute Force Detection and Audit Policy Gap](./labs/04-brute-force-detection.md) | Kali Linux, xfreerdp, hydra, EventCode 4625, audit policy, Atomic Red Team, EventCode 4720 | Complete |
| 05 | [Help Desk Simulation with Zendesk](./labs/05-zendesk-helpdesk.md) | Zendesk, ticketing workflow, account management, onboarding, offboarding, phishing analysis | Complete |
| 06 | [Metasploit Attack Chain and Remediation](./labs/06-metasploit-attack-remediation.md) | msfvenom, Metasploit, Meterpreter, CVE analysis, Splunk detection, incident response | Complete |

---

## Tools and Technologies

Windows Server 2022, Active Directory, Group Policy, DNS, DHCP, Windows 10, Ubuntu Server, Kali Linux, VirtualBox, Splunk, Sysmon, Universal Forwarder, Metasploit, msfvenom, Meterpreter, xfreerdp, Hydra, Atomic Red Team, Zendesk, PowerShell, Bash, RDP

---

## Attack Techniques Covered

Based on the MITRE ATT&CK framework:

- T1110 Brute Force
- T1136.001 Create Account: Local Account
- T1204 User Execution
- T1548 Abuse Elevation Control Mechanism
- T1562 Impair Defenses
- T1059 Command and Scripting Interpreter

---

## Certifications

- CompTIA Security+ (completed)
- AZ-900 Azure Fundamentals (in progress)
- AZ-104 Azure Administrator (up next)
