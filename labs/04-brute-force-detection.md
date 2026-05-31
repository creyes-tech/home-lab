# Lab 04 — Brute Force Attack Detection and Audit Policy Gap Analysis

**Date:** March 2026

**Category:** Threat Detection and Response

**Difficulty:** Intermediate

---

## Objective

Simulate a brute force RDP attack from Kali Linux against a domain-joined Windows 10 endpoint, detect the attack in Splunk, identify an audit policy gap that prevented full detection, fix the gap, and validate the fix using Atomic Red Team.

---

## Why This Matters

Brute force attacks against RDP are one of the most common attack vectors in the real world. Detecting them requires both the right logging configuration and the right monitoring in place. This lab demonstrates the full detection engineering cycle. Simulate an attack, check what the SIEM captured, identify what it missed, fix the logging gap, and confirm the fix works. This is exactly what a security engineer or SOC analyst does.

---

## Environment

| Component | Details |
|---|---|
| Attacker | Kali Linux, IP 192.168.10.250 |
| Target | TARGET Windows 10, IP 192.168.10.100 |
| SIEM | Splunk on Ubuntu, IP 192.168.10.10 |
| Tools | xfreerdp, hydra, Atomic Red Team |

---

## What I Built

**Simulated brute force RDP attack**

Enabled RDP on TARGET and added domain users bdoe and jsmith to the Remote Desktop Users group. From Kali ran repeated failed RDP connection attempts using xfreerdp with incorrect passwords to simulate a brute force attack followed by a successful connection.

**Detected the attack in Splunk**

Searched Splunk for EventCode 4625, which are Windows failed logon events. Found multiple entries showing failed authentication attempts originating from 192.168.10.250 targeting TARGET. Each event contained the attacker source IP, the targeted username, the workstation name, and the failure reason.

Key fields in the 4625 events:

- Source Network Address: 192.168.10.250 (attacker)
- Workstation Name: TARGET (victim)
- Account Name: jsmith (targeted account)
- Failure Reason: Unknown username or bad password

A burst of 4625 events from the same source IP in a short timeframe is the signature of a brute force attack.

**Identified audit policy gap**

Ran Atomic Red Team technique T1136.001 which simulates local user account creation, a common attacker persistence technique. Searched Splunk for EventCode 4720 which is user account created and found no results. The attack ran successfully but Splunk captured nothing.

Root cause: Windows does not log account management events by default. The audit policy for User Account Management was not enabled so Windows never wrote the event meaning the Splunk forwarder had nothing to collect.

**Fixed the audit policy**

On ADDC01 opened Group Policy Management, edited the Default Domain Policy, navigated to Computer Configuration, Policies, Windows Settings, Security Settings, Advanced Audit Policy Configuration, Audit Policies, Account Management. Enabled auditing for User Account Management for both Success and Failure.

**Troubleshot clock skew**

gpupdate /force failed with a clock synchronization error. Active Directory Kerberos authentication requires all domain members to be within 5 minutes of each other. The host laptop had the wrong date which propagated to the VMs. Corrected the host system clock which resolved the issue.

**Validated the fix**

Reran Atomic Red Team T1136.001. Searched Splunk for EventCode 4720 and confirmed the event appeared. Ran cleanup to remove the test account:

Invoke-AtomicTest T1136.001 -Cleanup

**Built a Splunk detection alert**

Created an alert that fires automatically when an executable runs from a Downloads folder.

Search: index=endpoint EventCode=4688 Image="*\\Downloads\\*.exe"

Settings: Real-time, triggers per result, High severity.

---

## Key Event Codes

| Event ID | Meaning | Why It Matters |
|---|---|---|
| 4625 | Failed logon | Burst of these from one IP indicates brute force |
| 4624 | Successful logon | After a burst of 4625s this indicates brute force succeeded |
| 4720 | User account created | Attacker persistence technique |
| 4688 | Process creation | Shows what executed, when, and by which user |
| 3 (Sysmon) | Network connection | Shows outbound connections made by specific processes |

---

## Decisions I Made and Why

**Why EventCode 4625 matters**

One failed logon is normal. Twenty in thirty seconds from the same source IP means someone is brute forcing an account. The pattern is what matters not any single event.

**Why the audit policy gap matters**

Before fixing the policy an attacker could have created a backdoor account on any machine in the domain and Splunk would have no record of it. Identifying and closing this gap means detection coverage is now complete for this technique.

**Why Atomic Red Team for validation**

Rather than guessing whether the fix worked Atomic Red Team provides a controlled repeatable simulation of the exact technique being tested. Run it, check Splunk, confirm the event appears. That is the correct way to validate detection coverage.

---

## Resources

- [MITRE ATT&CK T1136.001](https://attack.mitre.org/techniques/T1136/001/)
- [Atomic Red Team](https://github.com/redcanaryco/atomic-red-team)
- [Windows Security Event ID Reference](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/)
