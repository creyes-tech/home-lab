# Lab 02 — Active Directory Domain Setup

**Date:** March 2026

**Category:** Identity and Access Management

**Difficulty:** Intermediate

---

## Objective

Deploy and configure a fully functional Active Directory domain on Windows Server 2022 including domain promotion, DNS integration, organizational units, user accounts, and security groups.

---

## Why This Matters

Active Directory is the backbone of nearly every enterprise Windows environment. It controls authentication, authorization, and policy enforcement across every machine on the network. The majority of help desk tickets involve AD in some way including password resets, account lockouts, permission issues, new employee setup, and offboarding.

---

## Environment

| Component | Details |
|---|---|
| Domain Controller | ADDC01, Windows Server 2022 |
| Domain | tom.local |
| DNS | Integrated with AD on ADDC01 |
| Client | TARGET, Windows 10, domain joined |

---

## What I Built

**Renamed the server before promotion**

Learned the hard way that renaming a Domain Controller after promotion breaks Active Directory entirely. The machine name gets baked into the AD database during promotion and cannot be changed without rebuilding. The correct order is rename first then promote. Named the server ADDC01 before touching any roles.

**Installed Active Directory Domain Services**

Added the AD DS role through Server Manager then used the post-installation flag to promote the server to a Domain Controller. Selected Add a new forest with root domain name tom.local. Set forest and domain functional levels to Windows Server 2016. Configured DNS to install alongside AD since the Domain Controller needs to be the authoritative DNS server for the domain.

**Configured static IP on the Domain Controller**

Set static IP 192.168.10.7 on ADDC01 with DNS pointing to 127.0.0.1 meaning itself. A Domain Controller must have a static IP because every other machine needs to find it consistently, and it must use itself as DNS because it is the DNS authority for tom.local.

**Created Organizational Units**

Built an OU structure organized by department: IT, HR, Sales, and Finance under tom.local. OUs allow Group Policy to be applied at the department level so different rules can apply to different groups automatically.

**Created user accounts**

- bdoe (Bob Doe), standard user, Finance department
- jsmith (Jane Smith), standard user, HR department
- mscott (Michael Scott), standard user, Sales department

Tested domain login by signing into TARGET as TOM\bdoe and TOM\jsmith to confirm authentication worked correctly.

**Created Security Groups**

Created Finance-Team security group and added bdoe as a member. Created Sales-Mailbox security group and added bdoe as a member. Security groups control access to resources. Granting a group permission to a folder automatically grants access to every member of that group.

**Joined Windows 10 to the domain**

On TARGET went to Advanced System Settings, Computer Name, changed from Workgroup to Domain, entered tom.local. Initial attempt failed because DNS was still pointing to 8.8.8.8 instead of 192.168.10.7. Changed DNS in network adapter settings to 192.168.10.7, retried, and the machine joined successfully.

---

## What Broke

**Domain Controller rename broke AD**

Renamed the server after promotion which caused a trust relationship failure on every subsequent login. Error: the security database on the server does not have a computer account for this workstation trust relationship. Attempted DSRM recovery, netdom commands, and PowerShell. All failed. Rebuilt the Domain Controller from scratch with the correct name before promotion.

**Windows 10 could not find the domain**

When attempting to join tom.local the machine returned an Active Directory Domain Controller for the domain could not be contacted. Root cause was DNS on TARGET was set to 8.8.8.8 which has no knowledge of the private tom.local domain. Changed DNS to 192.168.10.7 and the domain join succeeded immediately.

**Clock skew breaking Group Policy**

gpupdate /force on TARGET failed with a clock synchronization error. Active Directory uses Kerberos authentication which requires all domain members to be within 5 minutes of each other. The host laptop had the wrong date which propagated to the VMs. Fixed the host clock which resolved the issue.

---

## Decisions I Made and Why

**Why point the DC DNS to itself**

A Domain Controller is the DNS server for its own domain. If it pointed DNS somewhere external it could not resolve its own domain records. 127.0.0.1 means ask yourself first which is correct for a DC.

**Why OUs instead of putting everything in Users**

OUs allow Group Policy to be scoped to specific departments. If Finance needs a different password policy than IT, OUs make that possible without affecting everyone. Putting everything in the default Users container means one policy applies to everyone which is not how real enterprises work.

**Why Security Groups for resource access**

Granting a security group access to a resource rather than individual users scales properly. When a new Finance employee joins you add them to Finance-Team and they automatically get access to everything that group has. When someone leaves you remove them and they lose all access instantly.

---

## Resources

- [Microsoft Active Directory Documentation](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/)
- [Kerberos Authentication Overview](https://docs.microsoft.com/en-us/windows-server/security/kerberos/kerberos-authentication-overview)****
