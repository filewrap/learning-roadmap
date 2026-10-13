# Cybersecurity: Windows and Active Directory

## What this subject is—and why it matters

Windows security study begins with how an individual computer represents identities, runs programs, and protects data. It then expands to Active Directory Domain Services (AD DS), a directory and identity service used by Windows domains. AD DS centrally organizes accounts, computers, groups, and policy, while domain controllers provide directory and authentication services. Because access to domain identities can affect many connected systems, understanding the relationship between a local machine and the directory is essential for both administration and security analysis.

This chapter follows a progression from Windows fundamentals to directory architecture, authentication, authorization, safe threat modeling, and defensive design. Its advanced topics—certificate services, delegation, trusts, and enterprise attack paths—are concepts to recognize and assess, not instructions for exploiting production environments. Microsoft's guidance emphasizes limiting privileged access, protecting domain controllers, and monitoring sensitive activity [3].

## Safe-lab boundary

Do hands-on work only in an intentionally vulnerable local lab, an official practice environment, systems you own, or systems covered by explicit written authorization. Use synthetic accounts and data. Do not probe real organizations, capture or reuse other people's credentials, dump secrets, relay authentication, evade detection, establish persistence, or make destructive changes. For the exercises below, inspect only your own machine or a purpose-built isolated lab; keep any directory changes reversible and documented.

## Prerequisites and vocabulary

You do not need to be a Windows administrator to begin. Useful foundations are basic command-line use, files and folders, IP networking (address, name resolution, client/server), and the difference between identifying a user and granting that user permission. Learn these terms as you go:

- **Identity / principal:** a user, computer, or service account that can be authenticated.
- **Authentication:** verifying who or what is requesting access. **Authorization** decides what an authenticated principal may do [1].
- **Local account:** an identity managed by one computer. **Domain account:** an identity managed by AD DS and usable across appropriately configured domain resources.
- **Group:** a named collection of principals, often used to grant a common set of permissions.
- **ACL (access control list):** rules attached to an object that describe which principals can perform which actions.
- **Privilege:** authority to perform a sensitive system or administrative operation; not the same thing as permission to read a particular file.
- **Protocol:** agreed rules for communication. **LDAP** is a directory-access protocol; **SMB** is commonly used for Windows file sharing; **RDP** provides remote desktop access.
- **Ticket:** a time-limited Kerberos credential used to request or prove access to a service. It is not simply a password [2].

## 1. Start with one Windows computer

### Architecture and security boundaries

A Windows system has a kernel (the privileged core that manages hardware, memory, and processes) and user-mode programs that request services from it. A **process** is a running program with its own execution context. A **service** is a program intended to provide a function, often in the background. A service's account and assigned rights matter: a bug in a highly privileged service can have greater consequences than one in an ordinary user's program.

The **registry** is a hierarchical configuration database used by Windows and applications. It is not a general-purpose file system, and changing a key can alter system behavior. Inspect before changing; record original settings and use a recoverable test machine.

### Users, groups, NTFS, and effective access

NTFS is a Windows file system that supports permissions on files and folders. Access depends on the security identity (represented internally by a security identifier), group membership, and the applicable permission rules. In practice, share permissions and file-system permissions can both apply to a network share; the effective outcome is constrained by both. Do not infer access merely from a displayed group name—check the actual object, inherited rules, and the account's group memberships.

A good mental model: authentication is showing an identity card; authorization is checking the list of doors that card can open. A successful sign-in does not imply administrator rights or permission to every resource. Windows access checks use a logon token (user and group SIDs, plus privileges) against an object security descriptor and the requested rights. Allow/deny entries, inheritance, and group nesting affect the outcome; a group listing alone is not a complete effective-access answer. For a network share, consider both share-level and NTFS controls, and which identity the connection actually uses.

### PowerShell as an inspection tool

PowerShell is a command shell and scripting language whose pipeline passes structured .NET objects, rather than only plain text [7]. This makes it useful for repeatable administration and read-only inventory. Learn to inspect command help and outputs before scripting changes. For example, in a PowerShell session on your own Windows computer, `Get-Process | Select-Object -First 5 ProcessName, Id` selects a small sample of running-process names and IDs. It reads information and does not modify the processes. Commands and available modules vary by Windows/PowerShell edition and installed features; verify the local help for your environment.

### Windows authentication and network services

Windows supports multiple authentication protocols and packages. In domain settings, Kerberos is the default domain protocol when conditions permit; NTLM remains relevant for compatibility and some configurations [1] [2]. A client obtains a TGT from the KDC, uses it to request a service-specific ticket, then presents that ticket to the service; ordinarily the service can validate it without asking a DC on every connection [2]. Therefore ticket caches are sensitive, and service naming/SPNs and domain discovery matter. A Kerberos failure does not by itself indicate a bad password: check name resolution, service naming, time synchronization, connectivity, and policy. **SMB** supports file sharing; signing provides message integrity, while encryption additionally protects traffic confidentiality. Encryption requires compatible SMB 3.x peers and may affect performance or legacy clients; test dependencies and scope before broad deployment [6]. These controls protect traffic, not files at rest. **RDP** is remote interactive access, not a substitute for sound identity controls: restrict who can use it and where it is reachable.

## 2. Active Directory: the shared directory

An AD DS **domain** is an administrative and replication boundary containing directory objects and domain resources. A **domain controller (DC)** is a server running AD DS; it stores directory data and provides services such as authentication. A **forest** is the top-level AD structure that can contain one or more domains with shared schema and configuration. Treat the DC as high-impact infrastructure: control administrative access and protect its operating environment [3].

Objects have names and attributes. LDAP provides operations such as bind (establishing an authenticated session), search, add, and modify; the protocol specification defines these request/response operations [5]. AD uses LDAP-compatible directory access, while AD-specific schema and behavior add Windows details. A **distinguished name (DN)** identifies an object's place in the directory hierarchy; an **organizational unit (OU)** is a container that can help structure administration and policy application.

**Group Policy** is a mechanism for centrally applying configured settings to users and computers. A Group Policy Object (GPO) links settings to directory scope; processing context (user or computer), site/domain/OU links, inheritance, filtering, and conflicting settings affect the result. A link is not proof that a setting applies: identify the target and validate resultant policy in an authorized environment. Test one scope first, check expected and unintended effects, and document rollback. **Shares** expose network resources, and their access is governed by identities, share settings, and underlying file permissions.

### Kerberos, in a simple flow

Windows Kerberos uses a Key Distribution Center (KDC), integrated into DCs. At sign-in, a client obtains a **ticket-granting ticket (TGT)**. When it needs a service, it asks the KDC for a service ticket, then presents that ticket to the service. The service can verify the ticket without contacting a DC for every connection [2]. The ticket system enables single sign-on, but tickets and their supporting keys are sensitive credentials.

A **Service Principal Name (SPN)** associates a service instance with an account so Kerberos can locate the correct service identity. **Delegation** permits a service to act on behalf of a user for another service in defined circumstances; it must be carefully constrained because it crosses service boundaries [2]. Learn the intended application flow before reviewing delegation settings.

**NTLM** is a challenge-response authentication family, still encountered in legacy or fallback paths [1]. It differs from Kerberos ticket-based authentication and has distinct security properties. Don't assume that a domain environment always used Kerberos: names, service configuration, connectivity, and policy can affect negotiation. Defenders should inventory and reduce legacy dependencies deliberately rather than disable authentication blindly.

## 3. From misconfiguration to risk: concepts, not attack recipes

Security findings are often about a chain of permissions, not a magic tool. For example, an ordinary account might have a path to modify a group, that group might administer a server, and that server might hold access to a more sensitive resource. Reviewing the whole chain reveals why **least privilege**—grant only rights needed for a role—is more useful than focusing only on well-known administrator groups [3]. Start from a protected asset and trace backward through identities, groups, delegated ACLs, services, and administrative paths. At each step ask who can grant or change access, whether it is needed, and how changes are detected; names alone are not evidence of effective access.

The repository plan names several attack categories. At a safe conceptual level:

- **Kerberoasting** concerns the risk that service-account authentication material associated with Kerberos service tickets can be subjected to offline password guessing when account secrets are weak. Defensive response: use long, unique managed service-account secrets, review service-account exposure, and monitor unusual ticket activity. This chapter intentionally omits collection and cracking procedures.
- **AS-REP roasting** refers to risk around accounts configured without Kerberos pre-authentication, which can expose material suitable for offline guessing. Review account configuration and remediate exceptions; do not test against accounts without authorization.
- **Pass-the-hash** and **pass-the-ticket** describe attempts to use captured NTLM-derived material or Kerberos tickets as authentication material. Treat hashes and tickets as credentials, minimize their exposure, limit privileged logons, and investigate unexpected use.
- **Credential dumping** is unauthorized extraction of secrets from a machine or directory. **Token impersonation** means abusing a process security token that carries identity and privileges. Defenses include reducing privileged sessions on untrusted hosts, endpoint protection, access controls, and audit review [3].
- **Lateral movement** means moving between systems after an initial foothold; **privilege escalation** means obtaining greater authority. These are outcomes to prevent and detect, not exercises to perform on live networks.

At the next level, **AD CS** (Active Directory Certificate Services) provides public-key infrastructure: it issues and manages digital certificates used for cryptography and signatures [4]. Certificate templates and enrollment permissions are security-critical: a certificate may assert an identity or purpose to relying systems. Review who can enroll, identity claims and key usages, private-key storage, approval requirements, issuance monitoring, and revocation lifecycle. Certificate-related risk depends on configuration and context; a template name alone does not prove vulnerability [4].

Other advanced directory topics are best understood as trust and replication boundaries:

- **Constrained delegation** limits which services a service may access on behalf of a user; **unconstrained delegation** is broader and carries greater exposure. **Resource-based constrained delegation (RBCD)** places control over which principals may act on behalf of users at the resource side. Review each configuration against the actual application need [2].
- A **Golden Ticket** is a forged TGT concept associated with compromise of the domain's Kerberos account key; a **Silver Ticket** refers to a forged service ticket using a service account key. Both imply severe key compromise, so response requires incident handling and key recovery planning, not merely deleting a suspicious file.
- **DCSync** abuses directory replication rights to request directory secrets as though performing replication. MITRE describes it as credential access and recommends controlling replication permissions [8]. **DCShadow** is a threat concept involving unauthorized directory changes through replication-like mechanisms. The defensive lesson is strict control of replication-capable rights and monitoring unexpected replication behavior.
- **NTLM relay** describes forwarding authentication exchanges to another service under vulnerable conditions; mitigate through protocol hardening, signing/channel protections where applicable, and reducing NTLM dependencies. **Advanced ACL abuse** means turning delegated object permissions into unintended control. **Forest/domain trust attacks** exploit trust relationships or weakly governed cross-boundary access. Map these permissions and trust paths before deciding what to change.

These names are useful vocabulary for reading incident reports and threat frameworks. They do not describe a universal exploit: feasibility depends on configuration, patches, protocol protections, privileges, and environmental context. Tools change quickly; the underlying identity, permission, key, and trust concepts change more slowly.

## 4. Safe practice tasks and answer checks

Use a personal Windows computer for read-only exercises, or a deliberately isolated lab for directory work. Do not run attack tooling or attempt to obtain credentials.

1. **Inventory a local machine.** Use built-in Windows Settings or read-only PowerShell commands to list local users/groups and a few running processes. **Success check:** you can distinguish a user from a group and a process from a service; you have made no configuration changes.
2. **Explain access.** Choose a test file you own. Inspect its security properties and identify its owner and listed permissions. **Success check:** describe why a successful logon alone does not establish read/write access; note that inherited rules or group membership can affect effective access.
3. **Map a fictional domain.** Draw `user → group → file share` and `client → DC/KDC → service` as two separate diagrams. **Success check:** the first expresses authorization; the second expresses authentication and ticket flow. Identify LDAP as directory access, SMB as file access, and Kerberos as authentication.
4. **Review a sample policy.** In a local lab or provided practice environment, inspect a harmless GPO setting and identify its intended scope. **Success check:** state which user/computer scope it targets and how you would verify application; make no changes unless the lab explicitly calls for a reversible one.
5. **Assess a fictional service account.** Given a made-up account with broad privileges and no documented owner, propose a defensive review. **Success check:** identify owner, business purpose, minimum required access, secret-management approach, logon scope, and monitoring before proposing removal or rotation.
6. **Read a directory-security scenario.** Given that an ordinary account has replication rights, explain why it is sensitive. **Success check:** connect replication rights to DCSync risk, recommend reviewing/limiting the ACL and monitoring replication events, and avoid attempting to retrieve secrets [8].
7. **Trace a fictional Kerberos failure.** A user can sign in but cannot access a named file service. List non-secret checks: DC discovery, service-name/SPN alignment, clock synchronization, and service reachability. **Success check:** distinguish domain authentication from service authorization; do not expose credentials or tickets.
8. **Review an admin design.** A domain administrator reads email on a workstation and administers DCs from it. Propose separate daily and privileged work, identify the highest-trust assets, and explain how to validate that lower-trust devices cannot administer them [3] [9].

## Common mistakes

- Treating domain membership as proof that every authentication uses Kerberos, or assuming NTLM is absent.
- Confusing authentication with authorization, or a user's visible name with the security identifier and group context used in access decisions.
- Assuming membership in one group tells the whole permission story; object ACLs, nesting, inheritance, share controls, and delegated rights may matter.
- Treating a ticket, hash, token, certificate, or service credential as harmless metadata. Each can be security-sensitive authentication material.
- Thinking “administrator” is one universal permission. Local administrator, domain privileges, delegated directory rights, and service-specific permissions are distinct.
- Copying a tool command from a CTF write-up into a real network. Tool behavior, protocol details, and authorization context matter; practice only in the safe boundary above.
- Making broad policy or protocol changes before mapping dependencies. Validate on a test scope, preserve recovery options, and monitor the outcome.

## Next steps

1. Build confidence with Windows accounts/groups, NTFS permissions, processes, services, registry concepts, and read-only PowerShell.
2. Learn networking and name resolution, then distinguish SMB, RDP, LDAP, Kerberos, and NTLM by purpose.
3. Build a small isolated AD lab or use an official practice environment; map domains, DCs, OUs, groups, GPOs, shares, and permissions.
4. Trace Kerberos sign-in and service-ticket flow, then study SPNs, service accounts, and delegation from a defensive configuration perspective.
5. Read AD CS concepts and review certificate issuance controls; study trust and replication boundaries.
6. Practice defensive reviews: least privilege, secure administration hosts, domain-controller protection, auditing, and recovery planning [3] [9].
7. Only then study named attack techniques as threat models, always paired with prevention, detection, and authorized lab validation.

## References

[1]: https://learn.microsoft.com/en-us/windows-server/security/windows-authentication/windows-authentication-overview "Windows authentication overview"
[2]: https://learn.microsoft.com/en-us/windows-server/security/kerberos/kerberos-authentication-overview "Kerberos authentication overview in Windows Server"
[3]: https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/best-practices-for-securing-active-directory "Best practices for securing Active Directory"
[4]: https://learn.microsoft.com/en-us/windows-server/identity/ad-cs/ "Active Directory Certificate Services documentation"
[5]: https://www.rfc-editor.org/rfc/rfc4511 "RFC 4511: Lightweight Directory Access Protocol (LDAP): The Protocol"
[6]: https://learn.microsoft.com/en-us/windows-server/storage/file-server/smb-security "SMB security"
[7]: https://learn.microsoft.com/en-us/powershell/scripting/overview "What is PowerShell?"
[8]: https://attack.mitre.org/techniques/T1003/006/ "OS Credential Dumping: DCSync"
[9]: https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/tier-model "Tier model for Active Directory Domain Services"
