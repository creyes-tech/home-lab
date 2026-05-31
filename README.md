Home Lab and Security Portfolio
Focus: Enterprise Windows Infrastructure | Threat Detection | Incident Response
Certs: CompTIA Security+ | In Progress: AZ-900 and AZ-104
GitHub: github.com/creyes-tech

Lab Overview
A virtualized enterprise attack and defense environment built entirely from scratch. Every entry documents what was built, what broke, the decisions made along the way, and what was learned. This is not a tutorial follow-along — it is a working lab that generates real attacks, real telemetry, and real incident response.
Domain: tom.local
Network: 192.168.10.0/24
VMOSIPRoleADDC01Windows Server 2022192.168.10.7Domain Controller, Active Directory, DNS, DHCPTARGETWindows 10192.168.10.100Simulated employee endpointSplunkUbuntu Server192.168.10.10SIEM, log collection and threat detectionKaliKali Linux 2025.4192.168.10.250Attack machine

Labs and Write-ups
#TitleSkillsStatus01Environment Build and TroubleshootingVirtualBox, VM deployment, NAT Network, static IPs, network troubleshootingComplete02Active Directory Domain SetupWindows Server 2022, AD DS, domain promotion, DNS, OUs, users, security groups, Group PolicyComplete03Splunk SIEM DeploymentUbuntu Server, Splunk Enterprise, Universal Forwarder, Sysmon, inputs.conf, log ingestionComplete04Brute Force Detection and Audit Policy GapKali Linux, xfreerdp, hydra, EventCode 4625, audit policy, Atomic Red Team, EventCode 4720Complete05Help Desk Simulation with ZendeskZendesk, ticketing workflow, account management, onboarding, offboarding, phishing analysisComplete06Metasploit Attack Chain and Remediationmsfvenom, Metasploit, Meterpreter, CVE analysis, Splunk detection, incident responseComplete

Tools and Technologies
Windows Server 2022 Active Directory Group Policy DNS DHCP Windows 10 Ubuntu Server Kali Linux VirtualBox Splunk Sysmon Universal Forwarder Metasploit msfvenom Meterpreter xfreerdp Hydra Atomic Red Team Zendesk PowerShell Bash RDP

Attack Techniques Covered (MITRE ATT&CK)

T1110 Brute Force
T1136.001 Create Account: Local Account
T1204 User Execution
T1548 Abuse Elevation Control Mechanism
T1562 Impair Defenses
T1059 Command and Scripting Interpreter


Certifications

 CompTIA Security+
 AZ-900 Azure Fundamentals — In progress
 AZ-104 Azure Administrator — Up next


Contact
github.com/creyes-tech
