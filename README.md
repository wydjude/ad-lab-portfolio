Building a Windows Server 2022 Active Directory Domain from Scratch
Overview

I built a complete Active Directory environment from a blank VM: installed Windows Server 2022, promoted it to a Domain Controller, configured DNS, designed an OU structure, provisioned test users and security groups, and enforced a security baseline through Group Policy: password complexity requirements and a legal login banner.

This project mirrors the first thing a sysadmin does when a business stands up new infrastructure, and it's the foundation almost every day-to-day service desk task (password resets, account unlocks, permission changes, GPO troubleshooting) sits on top of.

Environment
Component	Detail
Hypervisor	Oracle VirtualBox
Guest OS	Windows Server 2022 Standard Evaluation (Desktop Experience)
VM specs	4096 MB RAM, 2 vCPU, 50 GB dynamic disk
Domain name	lab.local
Static IP	192.168.1.10
Step 1: VM Provisioning

Created a new VM in VirtualBox, attached the Server 2022 evaluation ISO, and ran through a manual (not unattended) installation, selecting the Desktop Experience edition rather than Server Core so the environment stayed GUI-manageable for this stage of learning.

Note on a real fault hit along the way: an unattended-install attempt failed part-way through with a "Windows cannot find the Microsoft Software License Terms" error. Rather than troubleshoot the unattended installer, I deleted the VM and rebuilt it using a standard manual install. That was a judgement call about which failure is worth debugging versus which is faster to just redo.

Show Image Show Image Show Image

Step 2: Static IP Configuration

A Domain Controller can't run on a DHCP-assigned address, because every domain-joined machine needs to reliably find it at the same address every time to authenticate and resolve DNS. I set a fixed IP (192.168.1.10, subnet 255.255.255.0) and pointed the server's own DNS setting at itself (127.0.0.1), since this server is the DNS server for the domain.

Show Image

Step 3: Domain Controller Promotion + DNS

Installed the Active Directory Domain Services (AD DS) role via Server Manager, then ran the promotion wizard: Add a new forest, domain name lab.local.

Why DNS matters here: AD relies on DNS to function. It's not a separate, optional add-on. Domain-joined machines don't find the Domain Controller by IP; they query DNS for special service records (SRV records) that say "the DC for this domain lives here." That's why the DNS Server role installs automatically as part of promotion, running on the DC itself rather than as a separate per-server setup. One DNS service serves the whole domain.

After promotion, the server reboots and the login screen changes to reflect the domain (e.g. LAB\Administrator), confirming the machine is now a domain controller rather than a standalone server.

Show Image Show Image Show Image Show Image

Step 4: OU (Organizational Unit) Design

Built a nested OU structure: a parent OU (Corp) containing four child OUs: Users, Workstations, Groups, Servers.

Why OUs, and why nested: OUs exist to organise objects (users, computers, groups) so that administration and Group Policy can be scoped precisely, for example a password policy only applied to a specific department, without touching everyone in the domain. I nested mine under a parent Corp OU for two reasons: it's how a lot of real environments structure things (isolating custom, actively-managed objects from AD's built-in default containers), and, practically, I hit a naming collision trying to create a top-level Users OU, because AD already has a default container called Users sitting at that same level, and Windows won't allow both to share a name and parent. Nesting under Corp solved it cleanly and is arguably better practice anyway.

Show Image Show Image Show Image Show Image

Step 5: Users and Security Groups

Created 10 test user accounts inside the Users OU, and two security groups (IT-Support, Payroll) inside Groups, with members assigned to each.

Why groups matter: groups are how access and permissions get managed at scale. Rather than granting a shared drive or application access to ten individual people, you add them to a group and grant the group access once. Groups are also how Group Policy gets targeted: instead of applying a GPO to an entire OU, you can filter it to apply only to members of a specific security group, giving much finer control than OU structure alone.

A mistake I caught and fixed: I initially populated the test accounts with real names of people I know, before realising that was a problem for a portfolio screenshot going on a public GitHub repo, so I recreated the accounts with generic, clearly fictional names instead.

Show Image Show Image Show Image Show Image Show Image

Step 6: Group Policy: Password Policy and Login Banner

Created and linked a GPO (Domain Password Policy) at the domain level with two settings groups configured:

Password Policy: minimum length 12 characters, complexity requirements enabled.
Interactive logon banner: a message title and body text that displays before login, stating the system is for authorised use only.

Why this matters: password policy is a baseline security control. Enforcing minimum length and complexity domain-wide, rather than trusting individual users to choose strong passwords, closes off one of the most common attack vectors. The login banner is a genuine real-world requirement in a lot of regulated or security-conscious organisations: it's the legal notice that access is monitored and unauthorised use is prohibited, which matters if a company ever needs to take action against someone who accessed a system they shouldn't have.

Show Image Show Image Show Image Show Image

Challenges Encountered
Unattended Windows Server installation failed partway through. Resolved by rebuilding with a manual install instead of debugging the unattended path.
The VM kept re-running the OS installer on every reboot. This was caused by the installation ISO still being mounted with Optical set ahead of Hard Disk in the boot order. Fixed by ejecting the ISO (Devices > Optical Drives > Remove disk) and reordering the boot priority.
OU naming collision (Users) with AD's built-in default container, resolved by nesting custom OUs under a parent OU instead of creating them at the domain root.
Reflection: How This Applies on a Real Service Desk

Day-to-day, an entry-level support role is far more likely to involve using an AD environment someone else already built (resetting a password, unlocking an account, adding a user to a group, troubleshooting why a GPO isn't applying) than building a domain from scratch. But understanding what's actually happening underneath those tasks (why a static IP matters, why DNS is load-bearing for authentication, why GPOs get scoped through OUs and groups rather than applied blanket) is what separates being able to follow a script from being able to actually diagnose a problem when the standard fix doesn't work.

Skills Demonstrated
Windows Server 2022 installation and configuration
Active Directory Domain Services deployment and forest creation
DNS configuration in an AD environment
Organizational Unit design and delegation planning
Security group creation and membership management
Group Policy Object creation, linking, and configuration
Troubleshooting VM/hypervisor and unattended-installation faults
