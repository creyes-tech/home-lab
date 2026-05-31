# Lab 03 — Splunk SIEM Deployment and Log Ingestion

**Date:** March 2026

**Category:** Security Monitoring

**Difficulty:** Intermediate

---

## Objective

Deploy Splunk as a centralized SIEM on Ubuntu Server, install Splunk Universal Forwarder and Sysmon on all Windows endpoints, and configure log ingestion so security-relevant events from every machine flow into Splunk for analysis.

---

## Why This Matters

A SIEM is the central nervous system of a security operations center. Without one attacks happen silently. With a properly configured SIEM every meaningful event on every machine generates a log entry that can be searched, correlated, and alerted on in real time. Splunk is the industry standard and appears on almost every security and IT job posting.

---

## Environment

| Component | Details |
|---|---|
| Splunk Server | Ubuntu Server, IP 192.168.10.10 |
| Forwarder Endpoints | ADDC01 (192.168.10.7), TARGET (192.168.10.100) |
| Sysmon Config | olafhartong sysmon-modular |
| Log Index | endpoint |

---

## What I Built

**Deployed Splunk on Ubuntu Server**

Downloaded Splunk Enterprise .deb package on the native Windows host and transferred it to the Ubuntu VM via VirtualBox shared folder. Installed using dpkg. Configured Splunk to run as the splunk user and enabled boot-start so it starts automatically on reboot. Splunk web interface accessible at 192.168.10.10:8000 from any browser on the network.

**Configured receiving port**

In Splunk web interface configured a receiving port on 9997 under Settings, Forwarding and Receiving. This is the port that Universal Forwarders on Windows machines send logs to.

**Created endpoint index**

Created a dedicated index named endpoint to separate lab security logs from default Splunk data. All Windows event logs and Sysmon data routes to this index.

**Installed Sysmon on Windows endpoints**

Downloaded Sysmon from Microsoft Sysinternals and the olafhartong sysmon-modular configuration file. Installed on both ADDC01 and TARGET using:

sysmon64.exe -accepteula -i sysmonconfig.xml

Sysmon provides deep OS-level telemetry that standard Windows Event Logs do not capture including process creation with full command line arguments, network connections, file creation, and registry modifications.

**Installed and configured Splunk Universal Forwarder**

Installed Universal Forwarder on both ADDC01 and TARGET. During installation configured the deployment server as 192.168.10.10 port 9997. Created a custom inputs.conf file under the local directory:

[WinEventLog://Application]
index = endpoint
disabled = false

[WinEventLog://Security]
index = endpoint
disabled = false

[WinEventLog://System]
index = endpoint
disabled = false

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = endpoint
disabled = false
renderXml = true
source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational

Changed the Splunk Forwarder service to run as Local System account. This was required for it to have sufficient permissions to read the Security event log. Restarted the service after the change.

**Verified log ingestion**

Searched index=endpoint in Splunk and confirmed logs were flowing from both ADDC01 and TARGET. Two distinct hosts visible in search results confirming both forwarders were working.

---

## What Broke

**Splunk web interface unreachable from other VMs**

Root cause was the Ubuntu VM was on NAT while the other VMs were on NAT Network. These are completely isolated from each other in VirtualBox. Fixed by changing the Ubuntu VM network adapter to NAT Network with the same network name as all other VMs.

**Universal Forwarder not sending logs**

After installing the forwarder on TARGET searching Splunk showed no data from that host. Root cause was the inputs.conf file was placed in the default directory rather than the local directory. Splunk ignores default conf files. Custom configurations must go in the local directory. Moved the file to the correct location and restarted the service.

**Security log permission denied**

The Security event log requires elevated permissions to read. Fixed by changing the Splunk Forwarder service logon account to Local System in Windows Services.

**Shared folder not mounting on Ubuntu**

The VBoxGuestAdditions and vboxsf kernel module were not installed on Ubuntu. Installed virtualbox-guest-utils, added the user to the vboxsf group, rebooted, and the mount succeeded.

---

## Decisions I Made and Why

**Why a dedicated endpoint index**

Keeping security event data in a separate index makes searching faster and keeps data organized. In production Splunk environments there are often dozens of indexes for different data sources.

**Why olafhartong sysmon-modular config**

The default Sysmon installation with no config logs almost nothing useful. The sysmon-modular config is a community-maintained configuration that logs the right events without generating so much noise the SIEM becomes unusable. It is widely used in real security operations environments.

---

## Resources

- [Splunk Documentation](https://docs.splunk.com)
- [Sysmon Documentation](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [olafhartong sysmon-modular](https://github.com/olafhartong/sysmon-modular)
