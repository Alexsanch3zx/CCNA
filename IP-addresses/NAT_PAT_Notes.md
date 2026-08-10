# NAT / PAT — Theory Notes (CCNA)

**Audience:** CCNA students who need to understand **why address translation exists**, the four NAT address types, static vs dynamic vs **PAT (overload)**, and how to verify translations.

**Related files:** [terminology.md](../terminology.md) · [subnetting.md](subnetting.md) · [IPv4_vs_IPv6_Study_Guide.md](IPv4_vs_IPv6_Study_Guide.md) · [ACL-Study-Guide.md](../ACL-Study-Guide.md) · [command-cheat-sheet.md](../cheat-sheets/command-cheat-sheet.md) · [CCNA_Multi_Site_OSPF_NAT_Lab.md](../labs/Multi_Site_OSPF_NAT/CCNA_Multi_Site_OSPF_NAT_Lab.md) · [projects.md](../labs/projects.md)

---

## Part 1 — What Problem Does NAT Solve?

**NAT (Network Address Translation)** rewrites IP addresses (and often ports) as packets cross a boundary — typically **private LAN ↔ public Internet**.

Common reasons:

- **IPv4 scarcity** — many internal hosts share one (or a few) public addresses
- **Hide internal addressing** — outside hosts see the public address, not `192.168.x.x`
- **Overlap / migration** — translate between conflicting address spaces (advanced use)

**CCNA takeaway:** Inside hosts keep private RFC1918 addresses; the edge router translates traffic that leaves toward the Internet.

Private ranges reminder (`RFC 1918`):

| Range | CIDR |
| ----- | ---- |
| `10.0.0.0` – `10.255.255.255` | `10.0.0.0/8` |
| `172.16.0.0` – `172.31.255.255` | `172.16.0.0/12` |
| `192.168.0.0` – `192.168.255.255` | `192.168.0.0/16` |

---

## Part 2 — Inside / Outside and the Four Address Names

Cisco exams love this vocabulary. Draw it every time.

```text
[ Inside host ] --- (inside) --- [ NAT router ] --- (outside) --- [ Internet host ]
   192.168.1.10                      public IP                      203.0.113.10
```

| Term | Meaning | Example |
| ---- | ------- | ------- |
| **Inside local** | Real IP of the internal host (as seen on the LAN) | `192.168.1.10` |
| **Inside global** | Translated IP of that host **as seen by the outside** | `203.0.113.1` (or interface IP + port with PAT) |
| **Outside local** | Outside host’s IP **as seen from inside** (often same as outside global in simple labs) | `203.0.113.10` |
| **Outside global** | Real IP of the outside host | `203.0.113.10` |

**Memory trick for “inside local → inside global”:**

- **Local** = how the address looks **on your inside network**
- **Global** = how that same inside host looks **to the outside world**

---

## Part 3 — Types of NAT

### 3.1 Static NAT

One-to-one permanent mapping.

```text
Inside local 192.168.1.50  ↔  Inside global 203.0.113.50
```

Use when an internal server must be reachable from outside on a fixed public IP.

```cisco
ip nat inside source static 192.168.1.50 203.0.113.50
```

### 3.2 Dynamic NAT

Pool of public IPs; inside hosts get a temporary one-to-one mapping from the pool.

- Needs an **ACL** (who is allowed to translate)
- Needs a **pool** of global addresses
- If the pool is empty → new sessions fail

```cisco
access-list 1 permit 192.168.1.0 0.0.0.255
ip nat pool PUBLIC 203.0.113.10 203.0.113.20 netmask 255.255.255.0
ip nat inside source list 1 pool PUBLIC
```

### 3.3 PAT — Port Address Translation (NAT Overload)

Many inside hosts share **one** public IP. The router tracks flows with **TCP/UDP port numbers**.

```text
192.168.1.10:50100  →  203.0.113.1:50100
192.168.1.11:50100  →  203.0.113.1:50101   (port remapped if needed)
```

This is the **most common** home/SOHO and CCNA lab pattern.

```cisco
access-list 1 permit 192.168.1.0 0.0.0.255
access-list 1 permit 192.168.10.0 0.0.0.255

interface g0/0
 ip nat inside
!
interface s0/0/1
 ip nat outside
!
ip nat inside source list 1 interface s0/0/1 overload
```

**`overload`** = PAT.

**Lab mirror:** Same pattern as [CCNA_Multi_Site_OSPF_NAT_Lab.md](../labs/Multi_Site_OSPF_NAT/CCNA_Multi_Site_OSPF_NAT_Lab.md) Step 7.

---

## Part 4 — Interface Roles: `ip nat inside` / `ip nat outside`

NAT only works if interfaces are labeled correctly:

| Label | Typical placement |
| ----- | ----------------- |
| **`ip nat inside`** | LAN / VLAN / internal interfaces |
| **`ip nat outside`** | WAN / ISP-facing interface |

Traffic is translated when it moves **inside → outside** (for typical inside-source NAT).

**Common lab failure:** ACL and `overload` statement look perfect, but you forgot `ip nat inside` or `ip nat outside` on the right interfaces.

---

## Part 5 — The ACL’s Job in Dynamic NAT / PAT

The ACL does **not** “firewall the Internet” by itself in a NAT config.

For `ip nat inside source list <ACL> ...`, the ACL answers:

> **Which inside local addresses are allowed to be translated?**

| ACL match | Result |
| --------- | ------ |
| **permit** source | Eligible for NAT/PAT |
| **deny** / no match | Not translated by this rule (and often cannot reach outside as intended) |

Use a **standard ACL** (source only) for classic PAT labs — you care about **who** is inside, not destination details.

```cisco
access-list 1 permit 192.168.1.0 0.0.0.255
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.99.0 0.0.0.255
```

**Do not** put an unnecessary `deny any` mindset into NAT ACLs the same way you design security ACLs — remember NAT ACLs select candidates for translation. (Security filtering is a separate ACL story — see [ACL-Study-Guide.md](../ACL-Study-Guide.md).)

---

## Part 6 — Packet Walk (PAT Outbound)

Example: PC `192.168.1.10` pings `203.0.113.10` through HQ router doing PAT on `s0/0/1` (`10.0.0.1` in the multi-site lab).

1. PC sends packet: **src** `192.168.1.10`, **dst** `203.0.113.10`
2. Router receives on an **inside** interface
3. ACL matches → create/update translation
4. Router rewrites **src** to **inside global** (outside interface IP) and may change **src port**
5. Packet leaves **outside** interface toward ISP
6. Reply returns to inside global IP:port → router reverses the mapping → delivers to `192.168.1.10`

**Important:** Destination IP usually stays the same on this hop story; **source** is what PAT rewrites outbound.

---

## Part 7 — Static vs PAT vs “No NAT”

| Need | Use |
| ---- | --- |
| Many PCs share one public IP for outbound | **PAT / overload** |
| One internal server needs a fixed public IP | **Static NAT** (or static PAT/port forward concepts) |
| Enough public IPs for temporary 1:1 | **Dynamic NAT** pool |
| Pure private lab with no “Internet” edge | Often **no NAT** — just routing |

---

## Part 8 — Verification

```cisco
show ip nat translations
show ip nat statistics
clear ip nat translation *
```

| Command | What you learn |
| ------- | -------------- |
| `show ip nat translations` | Inside local ↔ inside global mappings (and ports for PAT) |
| `show ip nat statistics` | Hits, misses, pool usage, interface roles |
| `clear ip nat translation *` | Wipe dynamic entries (lab reset; disruptive) |

**After a successful ping from an inside PC to an outside host, you should see translations appear.**

Expected PAT flavor:

```text
Pro Inside global      Inside local       Outside local      Outside global
icmp 10.0.0.1:1        192.168.1.10:1     203.0.113.10:1     203.0.113.10:1
```

(Exact columns/ports vary; focus on **inside local → inside global**.)

---

## Part 9 — Order and Interaction With ACLs / Routing

Mental model for troubleshooting:

1. **Routing must work** conceptually — the router must know how to reach outside and return path must make sense
2. **Interface NAT direction** must be correct
3. **NAT ACL** must match the inside sources
4. **Security ACLs** (separate) can still block traffic even when NAT is fine

In the multi-site lab, Guest Internet is blocked with an **extended security ACL** on the Guest interface — that is **not** the same ACL used for PAT selection.

---

## Part 10 — IPv6 Note (Awareness)

IPv6 was designed with abundant addressing; classic NAT is an **IPv4** exam focus. You may hear **NAT64** / **NPTv6** in broader reading — not required at the same depth for CCNA PAT labs.

---

## Part 11 — Common Mistakes & Exam Traps

| Wrong idea | Correct idea |
| ---------- | ------------ |
| “NAT and PAT are unrelated” | **PAT is a type of NAT** (overload / many-to-one with ports) |
| “Inside global is the PC’s LAN IP” | Inside global is the **translated** address seen outside |
| “ACL for NAT filters destinations” | Standard NAT ACL usually selects **inside sources** allowed to translate |
| “Forgot overload but still call it PAT” | Without **`overload`**, you are not doing PAT on one interface IP |
| “Translations appear with no traffic” | Dynamic/PAT entries are created when matching traffic flows |
| “NAT replaces the need for a default route” | You still need correct **routing** to the outside |

---

## Part 12 — Practice Questions (Self-Check)

1. What does PAT use so many hosts can share one public IP?
2. Define **inside local** vs **inside global**.
3. Which interface command marks the LAN side of a NAT router?
4. What does the ACL do in `ip nat inside source list 1 interface g0/1 overload`?
5. Which command shows active translations?
6. Static NAT is best for what use case?

### Answers

1. **Port numbers** (and a translation table) — **overload**.
2. **Inside local** = host’s real private IP; **inside global** = how that host appears to the outside.
3. **`ip nat inside`**.
4. Selects **which inside local addresses** may be translated.
5. **`show ip nat translations`**.
6. A host/server that needs a **fixed one-to-one** public mapping.

---

## Part 13 — Quick Reference Card

```text
NAT rewrites addresses at the edge (private ↔ public)

Inside local  = real internal IP
Inside global = translated IP seen outside

Static NAT  = 1:1 permanent
Dynamic NAT = 1:1 from a pool
PAT/overload = many:1 using ports   ← most common

Needs:
  ip nat inside / ip nat outside
  ACL matching inside sources
  ip nat inside source list <ACL> interface <outside> overload

Verify:
  show ip nat translations
  show ip nat statistics
```

---

**Mastery check:** In the [multi-site OSPF/NAT lab](../labs/Multi_Site_OSPF_NAT/CCNA_Multi_Site_OSPF_NAT_Lab.md), ping the ISP host from an inside PC, then immediately run `show ip nat translations` and name each of the four address fields out loud.
