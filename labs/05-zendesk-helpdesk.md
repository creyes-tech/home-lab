# Lab 05 — Help Desk Simulation with Zendesk

**Date:** March 2026

**Category:** IT Operations

**Difficulty:** Beginner

---

## Objective

Build a simulated enterprise help desk environment using Zendesk, create realistic IT support scenarios, and work through ten tickets covering the most common help desk tasks, documenting every action the way a real IT professional would.

---

## Why This Matters

Technical skills alone are not enough in IT operations. Every action needs to be documented, tracked, and auditable. Ticketing systems are how IT departments manage workload, prove compliance, identify recurring problems, and communicate with users. Zendesk is one of the most widely used platforms in the industry.

---

## Environment

| Component | Details |
|---|---|
| Ticketing System | Zendesk |
| Fake Company | TOM Corp |
| End Users | bdoe (Bob Doe), jsmith (Jane Smith), mscott (Michael Scott), HR |
| AD Backend | All account changes performed on ADDC01 |

---

## Ticketing Workflow

Every ticket followed this documentation standard:

1. First internal note: what was reported and what will be investigated
2. Action: actual work performed in Active Directory or on the endpoint
3. Second internal note: what was found, what was done, outcome
4. Public reply: user-facing communication confirming resolution
5. Status: Open while working, Pending if waiting on user, Solved when complete

Internal notes are only visible to IT staff. Public replies go to the user. Troubleshooting details, account information, and internal reasoning stay internal.

---

## Tickets Completed

**Ticket 01: Account Lockout**

User bdoe reported being locked out. Checked AD account status and found it locked from failed login attempts. Unlocked the account and reset the password to a temporary credential. Sent user the temporary password and advised immediate change.

AD action: Properties, Account tab, unlock account. Reset Password.

**Ticket 02: New Employee Onboarding**

HR submitted a request to create an account for Michael Scott, Sales Manager. Created a Sales OU under tom.local. Created user account mscott in the Sales OU with Domain Users group membership. Replied to HR with username and login instructions.

**Ticket 03: Permission Denied**

User bdoe reported Access Denied accessing the Finance-Files shared folder. Investigated AD group membership and found bdoe was not a member of Finance-Team which has access to the folder. Added bdoe to Finance-Team and verified the group had correct permissions on the folder.

**Ticket 04: Software Installation Request**

User jsmith requested Adobe Acrobat Reader. Verified legitimate business need, approved the request, logged into TARGET as Administrator and installed it. Confirmed installation and notified user.

**Ticket 05: Password Reset**

User bdoe forgot his password, account was not locked. Reset the password in AD and communicated the temporary credential securely. Distinguished from Ticket 01 as no unlock was needed, only a reset.

**Ticket 06: Employee Offboarding**

HR notified IT that jsmith's last day was today. Immediately disabled the jsmith account in AD. Removed jsmith from all security groups. Added a note to the account description with the date and reason. Replied to HR confirming all access was revoked.

Offboarding tickets require the most thorough documentation due to compliance and legal implications. If an ex-employee retains access and causes damage the company needs proof IT acted immediately. The timestamp on a properly documented ticket is that proof.

**Ticket 07: Admin Access Request Denied**

User mscott requested local admin rights to install a sales tool. Denied the request as granting local admin to standard users violates security policy. A local admin can install any software, modify system settings, and bypass security controls. Offered to install the specific software on his behalf instead. Ticket set to Pending awaiting software details.

**Ticket 08: Shared Mailbox Access**

User bdoe requested access to the shared Sales inbox. Created a Sales-Mailbox security group in AD and added bdoe as a member. In a production environment this group would be linked to an Exchange or M365 shared mailbox.

**Ticket 09: Phishing Email Reported**

User jsmith forwarded a suspicious email claiming to be from Microsoft threatening account suspension in 24 hours. Analyzed the email and confirmed phishing. Indicators identified: sender domain was micros0ft-verify.net with a zero substituted for the letter o which is a typosquatting technique, artificial urgency with a 24 hour deadline, generic greeting instead of the user name, threatening language designed to override critical thinking, and Microsoft branding copy-pasted to appear legitimate. Confirmed user had not clicked any links. Advised deletion and explained indicators to watch for.

**Ticket 10: Proactive Account Audit**

Self-initiated quarterly review of all user accounts in the tom.local domain. Findings: bdoe active with appropriate group membership, jsmith disabled per Ticket 06 with all group memberships removed and confirmed, mscott active with appropriate membership, Administrator active for IT use only. No unauthorized or orphaned accounts found. Documented findings referencing the offboarding ticket for jsmith.

---

## Resources

- [Zendesk Documentation](https://support.zendesk.com)
