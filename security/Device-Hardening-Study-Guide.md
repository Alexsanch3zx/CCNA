# Device Hardening — AAA, Privilege Levels, Banners, Unused Services

**Audience:** CCNA students who need a practical checklist for **locking down routers and switches** — management access, least privilege, legal banners, and reducing attack surface.

**Folder:** `security/` — pairs with [Layer-2-Security-Study-Guide.md](Layer-2-Security-Study-Guide.md) (data-plane L2) and [../ACL-Study-Guide.md](../ACL-Study-Guide.md) (traffic filters).

**Related files:** [../terminology.md](../terminology.md) · [../cheat-sheets/command-cheat-sheet.md](../cheat-sheets/command-cheat-sheet.md) · [../labs/Multi_Site_OSPF_NAT/CCNA_Multi_Site_OSPF_NAT_Lab.md](../labs/Multi_Site_OSPF_NAT/CCNA_Multi_Site_OSPF_NAT_Lab.md) · [../CCNA-Networking-Study-Guide.md](../CCNA-Networking-Study-Guide.md)

---

## Part 1 — What “Hardening” Means

**Device hardening** = configure the device so that:

- Only **authorized** admins can manage it
- They get only the **privilege** they need
- Attackers get less to probe (no Telnet, no open unused services)
- Logs and banners support **accountability** and policy

This is mostly the **management plane** (how you configure the box), not the data plane (user traffic through the box).

**CCNA takeaway:** A perfectly routed network with Telnet + blank passwords is still a fail in production thinking.

---

## Part 2 — AAA Framework

**AAA** = **Authentication**, **Authorization**, **Accounting**.

| Letter | Question it answers |
| ------ | ------------------- |
| **Authentication** | Who are you? (password, username, certificate) |
| **Authorization** | What are you allowed to do? (privilege, commands, VLAN access) |
| **Accounting** | What did you do? (login/logout, command logging — conceptual) |

### 2.1 Local vs external AAA

| Method | Where credentials live | CCNA use |
| ------ | ---------------------- | -------- |
| **Local** | `username ... secret` on the device | Labs, small sites, backup method |
| **RADIUS** | External server | Common for network login / 802.1X |
| **TACACS+** | External server (Cisco-oriented) | Admin command authorization often preferred |

**Best practice idea:** Prefer central AAA in real networks; keep a **local fallback** admin for when the server is unreachable (carefully designed).

### 2.2 Local user (lab pattern)

```cisco
username admin privilege 15 secret CcnaLab2026!
enable secret CcnaLab2026!
```

Use **`secret`** (hashed) — not legacy `password` in clear context when you have a choice.

### 2.3 Line login tied to local users

```cisco
line console 0
 login local
 exec-timeout 5 0
!
line vty 0 4
 login local
 transport input ssh
 exec-timeout 5 0
```

| Piece | Why |
| ----- | --- |
| `login local` | Use local username database |
| `transport input ssh` | No Telnet on VTY |
| `exec-timeout` | Idle session drops (minutes seconds) |

### 2.4 AAA method lists (conceptual)

IOS can define ordered methods, e.g. try group RADIUS, then local:

```cisco
aaa new-model
aaa authentication login default group radius local
aaa authorization exec default group radius local
```

Exact lab depth varies — know **what AAA is** and that **method lists** pick the order of servers vs local.

---

## Part 3 — Privilege Levels

Cisco IOS privilege levels run from **0–15**.

| Level | Typical meaning |
| ----- | --------------- |
| **1** | User EXEC (`>`) — limited show/ping |
| **15** | Privileged EXEC (`#`) — full config access |
| **2–14** | Customizable intermediate levels (less common in intro labs) |

```cisco
username junior privilege 1 secret JuniorPass
username admin privilege 15 secret AdminPass

enable secret StillNeededForSomeFlows
```

**`enable` / `enable secret`:** Moves a session from user EXEC toward privileged EXEC when authorization model requires it.

**Least privilege:** Do not give every intern privilege 15. In real ops, TACACS+ command authorization is cleaner than hand-building levels 2–14 — for CCNA, know **1 vs 15** cold.

### Privilege vs role (mental model)

```text
Authentication → you proved identity
Authorization / privilege → you may enter config mode or not
```

---

## Part 4 — Banners

Banners display a message on login. They are not encryption — they are **policy / legal notice**.

| Type | When it shows | Common use |
| ---- | ------------- | ---------- |
| **MOTD** | Message of the day (often before login) | Warning / authorized use only |
| **Login** | Before username prompt | Legal banner |
| **Exec** | After successful login | Reminders |

```cisco
banner motd # Unauthorized access prohibited. Activity may be monitored. #
banner login # Authorized users only. #
```

Delimiter is a character you choose (`#` here) that does not appear in the message.

**Exam / ops tip:** A friendly “Welcome!” banner can weaken legal cases. Prefer a clear **unauthorized access prohibited** style.

---

## Part 5 — Secure Remote Access (SSH Baseline)

Hardening almost always includes **SSH instead of Telnet**.

```cisco
ip domain-name lab.local
crypto key generate rsa modulus 2048
ip ssh version 2

username admin privilege 15 secret YourPassword

line vty 0 4
 login local
 transport input ssh
```

Optional: restrict management sources with an ACL:

```cisco
ip access-list standard MGMT
 permit 192.168.1.0 0.0.0.255
 deny any
!
line vty 0 4
 access-class MGMT in
```

**Lab mirror:** SSH + VTY pattern in [CCNA_Multi_Site_OSPF_NAT_Lab.md](../labs/Multi_Site_OSPF_NAT/CCNA_Multi_Site_OSPF_NAT_Lab.md) Step 9.

Also set:

```cisco
service password-encryption    ! weak reversible obfuscation for some legacy password types — still better than clear in show run
no ip http server
no ip http secure-server       ! if you are not using device GUI
```

---

## Part 6 — Disable / Limit Unused Services

Every open service is attack surface. CCNA-level examples:

| Service / feature | Hardening action |
| ----------------- | ---------------- |
| **Telnet** | `transport input ssh` (do not allow telnet) |
| **HTTP/HTTPS server** | `no ip http server` / disable secure server if unused |
| **Unused interfaces** | `shutdown` |
| **CDP on internet-facing edges** | Consider `no cdp enable` on exposure ports (trade-offs inside LAN) |
| **Unused VTY lines** | Limit line range; ACL with `access-class` |
| **Pad / small services** (legacy) | Disable obscure IP services where present on older IOS (`no service tcp-small-servers`, etc.) |

Physical / console:

```cisco
line console 0
 login local
 exec-timeout 5 0
```

**Unused physical ports on switches:** `shutdown` + sometimes a “parking” VLAN — reduces accidental DHCP/rogue device on live access VLANs (pairs with L2 security notes).

---

## Part 7 — Passwords and Secrets (Quick Hygiene)

| Practice | Why |
| -------- | --- |
| `enable secret` | Stronger hashing than old `enable password` |
| Unique local admin secrets | Shared “cisco/cisco” fails interviews and audits |
| Different enable vs user secrets | Layered credentials |
| Do not put production secrets in public Git | Use lab-only passwords in this repo |

---

## Part 8 — Logging and Time (Hardening Adjacent)

Hardened devices should produce trustworthy logs:

```cisco
service timestamps log datetime msec
logging host 192.168.1.50
ntp server 192.168.1.50
```

Without time sync, incident timelines are mush. (Deeper NTP/syslog notes can be a separate file later.)

---

## Part 9 — Minimal Hardening Checklist (Lab → Prod Mindset)

1. Set **hostname** and **domain** (SSH keys need domain)
2. **`enable secret`** + local **username** with appropriate privilege
3. **Banner** (motd/login)
4. Console + VTY: **`login local`**, **`exec-timeout`**
5. VTY: **`transport input ssh`** only
6. Generate RSA keys; **SSH v2**
7. Optional **VTY ACL** (`access-class`)
8. **`shutdown`** unused interfaces
9. Disable **HTTP** if unused
10. Save: `write memory` / `copy run start`

---

## Part 10 — Common Mistakes & Exam Traps

| Wrong idea | Correct idea |
| ---------- | ------------ |
| “AAA is only RADIUS” | AAA is the **framework**; local is still AAA components |
| “Privilege 15 means no enable secret ever” | Models differ; know both **user privilege** and **enable** |
| “Banner blocks hackers” | Banner is **notice**, not access control |
| “Password encryption = strong crypto” | `service password-encryption` is **weak obfuscation** |
| “Hardening = ACLs on user VLANs only” | Also lock **management plane** (VTY/SSH/services) |

---

## Part 11 — Practice Questions (Self-Check)

1. Expand AAA and state what each letter means in one phrase.
2. What privilege level is full admin on IOS?
3. Why prefer `transport input ssh` on VTY lines?
4. What is the purpose of a login/MOTD banner?
5. Name two unused services/features you might disable on a hardened device.
6. What does `login local` do on a line?

### Answers

1. **Authentication** (who), **Authorization** (what allowed), **Accounting** (what happened).
2. **15**.
3. Telnet sends credentials in clear text; SSH encrypts management traffic.
4. Legal/policy warning — authorized use only / monitoring notice.
5. Examples: **Telnet**, **HTTP server**, unused **interfaces**, unnecessary small services.
6. Authenticates using the device’s **local username** database.

---

## Part 12 — Quick Reference Card

```text
Hardening = protect management plane + reduce attack surface

AAA = AuthN / AuthZ / Accounting
  local users OR RADIUS / TACACS+

Privilege 1 = user EXEC    Privilege 15 = full admin
enable secret + username ... privilege N secret ...

Banner = legal notice (motd / login)
SSH only on VTY; exec-timeout; optional access-class ACL
Shutdown unused interfaces; disable unused HTTP/Telnet

Checklist: secrets → users → banner → console/VTY → SSH → ACLs → no unused services → save
```

---

**Mastery check:** Take any lab router and apply the 10-step checklist from memory, then prove login works only via SSH with your local admin user — that is device hardening in practice.
