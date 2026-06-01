# Active Directory Study Guide

### On-Premises AD DS with Hybrid & Cloud Identity Context

**Audience:** People breaking into IT who need **Windows Server Active Directory Domain Services (AD DS)** for support, administration, or hybrid roles—and who want a structured map of concepts, operations, and verification.

**Companion contexts:** Modern jobs often pair on‑prem AD with **Microsoft Entra ID** (formerly Azure AD). This guide emphasizes **AD DS** first, then connects to **hybrid identity** where it matters day to day.

---

## How to Use This Guide

1. Read **Parts 1–4** for core architecture; **Parts 5–8** for daily admin and troubleshooting.
2. For each topic, practice: **where it lives in GUI**, **PowerShell/cmd equivalents**, and **what breaks when misconfigured**.
3. Lab with **evaluation VMs** (Windows Server + Windows clients), or employer/school tenants **only with permission**.

---

## Part 1 — What Active Directory Is

### 1.1 Definition

**Active Directory Domain Services (AD DS)** is Microsoft’s **directory service**: a hierarchical database of objects (**users, groups, computers, printers, etc.**), secured by **Kerberos** and **LDAP**, replicated among **domain controllers (DCs)**.

### 1.2 Why Organizations Use It

- **Central sign-on** for domain-joined Windows clients and many apps.  
- **Group Policy** for configuration and security baselines.  
- **Delegation** of administration via OU structure and permissions.  
- Integration with **DNS**, **DNS dynamic updates**, **certificate services**, **file servers**, and **Exchange / legacy Microsoft workloads**.

### 1.3 AD DS vs Other “Active Directory” Names


| Name                    | Meaning                                                                                            |
| ----------------------- | -------------------------------------------------------------------------------------------------- |
| **AD DS**               | On‑prem domain controllers, LDAP/Kerberos, classic “domain join”.                                  |
| **Entra ID / Azure AD** | Cloud directory; **not** a full LDAP DC in the traditional sense (different protocols and models). |
| **Azure AD DS**         | Managed domain service in Azure—similar *experience*, different ops model.                         |
| **LDAP**                | Protocol many directories use; AD DS is an LDAP directory with Microsoft schema and Kerberos auth. |


---

## Part 2 — Logical Structure

### 2.1 Forest, Tree, Domain

- **Forest** — top boundary; **one schema** and **global catalog** topology; **trust** boundary for security decisions at forest level.  
- **Tree** — contiguous DNS namespace (`corp.example.com` → `na.corp.example.com`).  
- **Domain** — primary administrative/security boundary for most policies; holds objects and DCs.

**Rule of thumb:** Multiple domains in one forest share a schema and GC; **trusts inside a forest are transitive by default**.

### 2.2 Organizational Units (OUs)

- **Containers** for delegation and **Group Policy** linkage—not security principals.  
- Design for **admin delegation** and **GPO scope**, not for mimicking org charts unless that helps ops.

### 2.3 Default Containers vs OUs

- `**CN=Users`** and `**CN=Computers`** are default containers—**GPOs linked to domain apply**, but many orgs move objects to **OUs** for structured policy.

### 2.4 Trusts (Overview)

- **Transitive** trusts propagate within forest; **external trusts** can be non‑transitive.  
- **Forest trusts** link forests for cross‑forest authentication (design-sensitive).

---

## Part 3 — Physical & Topology Concepts

### 3.1 Domain Controllers

- Hold writable or read-only copies of the directory (**GC**, **RODC** concepts below).  
- **Must be highly available**; losing all DCs for a domain is catastrophic without recovery paths.

### 3.2 Global Catalog (GC)

- Partial attribute set for **objects across the forest**; enables **forest-wide logon** and some lookups.  
- Each DC can be GC or not; typically **every site has a GC** for login performance.

### 3.3 Read-Only Domain Controllers (RODC)

- **One-way replication** of secrets to branch scenarios; limits credential exposure on less trusted sites.

### 3.4 Sites, Subnets, and Replication

- **Sites** map to **well-connected network areas** (usually physical locations).  
- **Subnets** associate IP ranges with sites so DC locator and clients find **local DCs**.  
- **Inter-site replication** is cost-controlled (ISTG/bridgeheads—know they exist); **intra-site** is more frequent.

**Symptom:** Logons slow cross‑WAN → often **wrong site/subnet** association or missing **GC** locally.

### 3.5 Operations Masters (FSMO Roles)

Five flexible single master roles—**only one DC holds each role at a time** in scope:


| Role                      | Scope  | Purpose (concise)                                            |
| ------------------------- | ------ | ------------------------------------------------------------ |
| **Schema Master**         | Forest | Schema updates                                               |
| **Domain Naming Master**  | Forest | Domain add/remove in forest                                  |
| **PDC Emulator**          | Domain | Time sync hub, password urgent replication, legacy semantics |
| **RID Master**            | Domain | Relative ID blocks for SIDs                                  |
| **Infrastructure Master** | Domain | Cross‑domain reference updates (unless GC—special cases)     |


**Seizing vs transferring:** transfer during graceful moves; **seize** only when partner DC is dead—**authoritative** recovery implications.

---

## Part 4 — Objects & Identity

### 4.1 Users

- **SAM account name**, **UPN** (`user@domain`), **DN**.  
- Account flags: **disabled**, **locked**, **password expired**, **cannot change password**, etc.

### 4.2 Groups


| Scope            | Typical use                                           |
| ---------------- | ----------------------------------------------------- |
| **Domain Local** | Permissions on resources *in that domain*             |
| **Global**       | Users from same domain; AGDLP pattern building blocks |
| **Universal**    | Mail‑enabled / forest-wide membership where needed    |


**AGDLP (classic pattern):** Accounts → **Global** groups → **Domain Local** groups → **Permissions** on resources.

### 4.3 Computers

- Machine accounts authenticate like users (Kerberos); **password rotation** on schedule.  
- **Secure channel** breaks → symptoms include trust failures; `**Test-ComputerSecureChannel`** (PowerShell) or `**nltest`** for diagnosis.

### 4.4 Security Identifiers (SIDs)

- Every principal has a **SID**; permissions store **SIDs**, not display names—rename-safe.

---

## Part 5 — Authentication & Protocols

### 5.1 Kerberos (Conceptual Depth)

- **Ticket Granting Ticket (TGT)** from **KDC** (runs on DC).  
- **Service tickets** for servers/apps; **SPNs** identify services (critical for duplicates breaking apps).  
- **Delegation** (constrained/unconstrained)—high‑risk when loose.

### 5.2 NTLM (Legacy)

- Still appears for **non‑Kerberos** scenarios; weaker; audit and reduce where possible.

### 5.3 LDAP

- **LDAP binds**, searches (`base DN`, `filter`, `attributes`).  
- Tools: **AD Users & Computers** advanced features, **ADSI Edit**, **ldapsearch**, **ldp.exe**.

---

## Part 6 — DNS Integration

### 6.1 Why DNS Is Non‑Negotiable

- Clients locate DCs via **SRV records** (e.g., `_ldap._tcp.dc._msdcs.domain`).  
- **Dynamic registration** by DCs and domain members when configured.

### 6.2 Common Failures

- Wrong DNS on clients → **cannot join domain**, **logon failures**, **slow GP**.  
- Non‑Microsoft DNS is fine **if** SRV records and updates are correct.

---

## Part 7 — Group Policy (GPO)

### 7.1 Mechanics

- **GPOs** linked to **site, domain, or OU**; **LSDOU precedence** (Local → Site → Domain → OU); **enforced** and **block inheritance** change effective order.  
- **Security Filtering**, **WMI filters**, **item-level targeting** (preferences).

### 7.2 Major Categories

- Security settings (password policy scope nuances with **fine‑grained password policies**).  
- Scripts, preferences (drive maps, shortcuts), Windows settings.  
- Software deployment (less common now vs modern management).

### 7.3 Troubleshooting Toolkit

- `**gpresult /h`**, **Group Policy Results** wizard, **event logs** on client and DC.  
- Check **network**, **DNS**, **SYSVOL/DFS-R replication** when policies don’t arrive.

---

## Part 8 — Administration Tools

### 8.1 MMC Snap-Ins & Consoles

- **Active Directory Users and Computers (dsa.msc)**  
- **Sites and Services (dssite.msc)**  
- **Domains and Trusts (domains.msc)**  
- **Group Policy Management (gpmc.msc)**

### 8.2 PowerShell Module — **ActiveDirectory**

Examples you should recognize (syntax varies by environment):

```powershell
Get-ADDomain
Get-ADUser -Filter * -Properties LastLogonDate
New-ADGroup -Name "SG-Example" -GroupScope Global -GroupCategory Security
Move-ADObject -Identity <DN> -TargetPath "OU=Sales,DC=corp,DC=local"
gpupdate /force   # run on client
```

### 8.3 RSAT

**Remote Server Administration Tools** on Windows clients enable the MMC/GPMC tools without logging on to a DC.

---

## Part 9 — High Availability & Recovery Concepts

### 9.1 Backups

- **System State** backup captures AD on that DC; understand **authoritative vs non‑authoritative restore** scenarios.  
- Modern vendors align with Microsoft backup APIs—know your org’s **RPO/RTO**.

### 9.2 Disaster Recovery Realities

- **Duplicate restored DCs** without proper isolation cause **USN rollback** risks—follow Microsoft procedures for **virtualization safeguards** and DC cloning rules.

---

## Part 10 — Monitoring & Security Baselines

### 10.1 Audit & Logs

- **Security event log** on DCs: logon events, Kerberos issues, changes to sensitive groups.  
- **Advanced Audit Policy** for granularity.

### 10.2 Tiered Administration (Concept)

- **Tier 0** = domain controllers and accounts that control them; strict isolation reduces lateral movement.

### 10.3 Common Attack Patterns (Awareness)

- **Kerberoasting** (service account weak passwords), **DCSync**, **Pass-the-Hash** mitigations via **credential hygiene**, **PAWs**, **Privileged Access Workstations**, **JIT/JEA**—depth belongs in security specialization, but **know the vocabulary**.

---

## Part 11 — Hybrid Identity (Entra Connect Context)

### 11.1 Why Hybrid Exists

- Same users sign in to **cloud** and **on‑prem**; devices and apps vary.

### 11.2 Sync & Auth Models (High Level)

- **Password Hash Sync**, **Pass-through Authentication**, **federated (AD FS)**—trade-offs on availability and control.  
- **Microsoft Entra Connect** synchronizes objects and handles matched identities.

### 11.3 Devices

- **Hybrid Azure AD join** vs **Entra join**—different management stacks (**Intune**, Group Policy, etc.).

---

## Lab Checklist (Skills to Prove Hands-On)

1. Promote a server to DC; verify **DNS SRV** records.
2. Join a client; confirm `**nltest /dsgetdc`** and `**whoami /groups`**.
3. Create **OUs**, **users**, **groups**; implement **AGDLP** on a shared folder with **NTFS + share** permissions.
4. Build and troubleshoot a **GPO** (e.g., mapped drive via Preferences + security filtering).
5. Define **sites/subnets**; observe **DC locator** behavior when moving a client subnet.
6. Simulate DNS failure on a client and restore correct resolution.
7. Inspect **Replication** with `**repadmin`** (basic `/replsummary`, `/showrepl`).
8. Optional: deploy **Entra Connect** in an isolated lab with Microsoft’s guidance only—never against production without authorization.

---

## Troubleshooting Flow (First-Line)

1. **Scope:** One user, one machine, one site, or everyone?
2. **DNS:** Client DNS points to DCs or valid internal resolvers with AD zones?
3. **Time skew:** Kerberos breaks beyond skew tolerance—**PDC emulator** hierarchy matters.
4. **Secure channel / trust:** Computer password, domain membership.
5. **Replication:** Recent changes missing—check **latency**, **bridges**, **firewalls** between DCs.
6. **GP:** `gpresult`, RSOP, SYSVOL health.

---

## Certification & Role Mapping (Orienting Only)

- **Microsoft Certified: Identity and Access Administrator Associate (SC-300)** emphasizes **Entra ID**, governance, hybrid—pair this guide’s **AD DS** sections with official SC-300 skills measured.  
- **Windows Server** specialty exams evolve—always read Microsoft’s **current** exam outline before scheduling.

---

## Appendix — Quick Term Glossary


| Term       | Meaning                                     |
| ---------- | ------------------------------------------- |
| **DN**     | Distinguished Name—full LDAP path to object |
| **UPN**    | User Principal Name—often looks like email  |
| **GC**     | Global Catalog                              |
| **RODC**   | Read-Only Domain Controller                 |
| **KDC**    | Key Distribution Center (Kerberos on DC)    |
| **SPN**    | Service Principal Name                      |
| **SYSVOL** | GPO/scripts replication share               |


---

**Practice ethically.** Only use domains and tenants you own or are explicitly authorized to test.

Good luck building identity skills—they underpin most enterprise IT careers.