# Access Control Lists (ACLs) — Theory Notes (CCNA)

**Audience:** CCNA students who need to understand **standard vs extended ACLs**, **inbound vs outbound** direction, **named vs numbered** lists, wildcard masks, and the **implicit deny**.

**Related files:** [terminology.md](terminology.md) · [IP-addresses/subnetting.md](IP-addresses/subnetting.md) · [IP-addresses/NAT_PAT_Notes.md](IP-addresses/NAT_PAT_Notes.md) · [inter_vlan_routing.md](inter_vlan_routing.md) · [cheat-sheets/command-cheat-sheet.md](cheat-sheets/command-cheat-sheet.md) · [labs/CCNA_Small_Office_Network_Lab.md](labs/CCNA_Small_Office_Network_Lab.md) · [labs/ACL_Router-on-a-stick_VLAN/CCNA_ACL_Lab.md](labs/ACL_Router-on-a-stick_VLAN/CCNA_ACL_Lab.md) · [labs/ACL_Port_Security/CCNA_ACL_Port_Security_Lab.md](labs/ACL_Port_Security/CCNA_ACL_Port_Security_Lab.md) · [labs/CCNA_Multi_Site_OSPF_NAT_Lab.md](labs/CCNA_Multi_Site_OSPF_NAT_Lab.md)

---

## Part 1 — What Is an ACL?

An **Access Control List (ACL)** is an ordered set of **permit** / **deny** statements that a router (or L3 switch) uses to **match packets** and decide whether to allow them.

CCNA uses ACLs for:

- **Traffic filtering** (security policy between VLANs/subnets)
- **NAT candidate selection** (which sources may translate — different intent)
- **Route filtering** / other feature matching (awareness)

This file focuses on **packet-filtering ACLs** on interfaces (`ip access-group`).

**CCNA takeaway:** ACLs are processed **top to bottom**. First match wins. If nothing matches, **implicit deny all**.

---

## Part 2 — How Matching Works

1. Packet arrives at an interface where an ACL is applied in a direction (`in` or `out`)
2. Router checks statements **in order**
3. **First match** → apply that permit/deny → **stop**
4. No match → **deny** (invisible rule at the end)

```text
deny   Sales → HR
permit any any
(implicit deny any)   ← still there, but unreachable if permit any any exists
```

**Design rule:** Put **more specific** matches **above** broader ones.

---

## Part 3 — Standard vs Extended ACLs

### 3.1 Standard ACL

Matches **source IP only**.

| Property | Detail |
| -------- | ------ |
| Numbered ranges | **1–99**, **1300–1999** |
| Matches | Source address (and wildcard) |
| Cannot match | Destination IP, protocol, port |

```cisco
access-list 10 permit 192.168.10.0 0.0.0.255
access-list 10 deny any
```

**Placement guideline (classic teaching):** apply standard ACLs **as close to the destination as possible** — because they cannot be precise about where the packet is going, filtering too early can block unintended destinations.

### 3.2 Extended ACL

Matches **source, destination, protocol, and ports** (and more).

| Property | Detail |
| -------- | ------ |
| Numbered ranges | **100–199**, **2000–2699** |
| Matches | Protocol, source, destination, ports, etc. |

```cisco
access-list 100 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
access-list 100 permit ip any any
```

**Placement guideline:** apply extended ACLs **as close to the source as possible** — you can be precise, so you drop bad traffic early.

**Lab mirror:** Small office lab blocks Sales → HR with extended ACL `100` on `g0/0.20 in`.

---

## Part 4 — Numbered vs Named ACLs

| Style | Example | Pros |
| ----- | ------- | ---- |
| **Numbered** | `access-list 100 deny ...` | Short; traditional exam style |
| **Named** | `ip access-list extended GUEST-FILTER` | Readable; easier to edit line-by-line |

### Named extended example

```cisco
ip access-list extended GUEST-FILTER
 deny ip 192.168.99.0 0.0.0.255 any
 permit ip any any
exit

interface g0/2
 ip access-group GUEST-FILTER in
```

**Lab mirror:** [CCNA_Multi_Site_OSPF_NAT_Lab.md](labs/CCNA_Multi_Site_OSPF_NAT_Lab.md) Step 8 (`GUEST-FILTER`).

### Named standard example

```cisco
ip access-list standard MGMT-ONLY
 permit 192.168.1.0 0.0.0.255
 deny any
```

**Editing tip:** Named ACLs let you delete/resequence individual lines more sanely than old numbered ACLs (platform-dependent). Prefer **named** in real configs for clarity.

---

## Part 5 — Wildcard Masks (Not Subnet Masks)

ACLs use **wildcard masks**, not subnet masks.

| Bit in wildcard | Meaning |
| --------------- | ------- |
| **0** | Must **match** this bit |
| **1** | **Ignore** this bit (“don’t care”) |

Common patterns:

| Match this | Wildcard |
| ---------- | -------- |
| Host `192.168.10.5` | `0.0.0.0` (or use `host 192.168.10.5`) |
| `/24` subnet `192.168.10.0` | `0.0.0.255` |
| `/16` `192.168.0.0` | `0.0.255.255` |
| `/30` `10.0.0.0` | `0.0.0.3` |
| Any | `any` (same idea as `0.0.0.0 255.255.255.255`) |

**Quick convert:** wildcard ≈ `255.255.255.255` **minus** subnet mask.

Example: mask `255.255.255.0` → wildcard `0.0.0.255`.

See also [IP-addresses/subnetting.md](IP-addresses/subnetting.md) and binary practice in [binary-hexadecimal/Binary-and-Hex-for-Networking.md](binary-hexadecimal/Binary-and-Hex-for-Networking.md).

---

## Part 6 — Direction: `in` vs `out` (The Hard Part)

```cisco
interface g0/0.20
 ip access-group 100 in
```

Direction is always **from the router’s point of view on that interface**:

| Direction | Meaning |
| --------- | ------- |
| **`in`** | Packets **entering** the router **through this interface** |
| **`out`** | Packets **leaving** the router **out this interface** |

### Worked example — Small Office lab

Goal: **Sales (VLAN 20) cannot reach HR (VLAN 10)**; HR can still reach Sales.

- Sales PCs use gateway `g0/0.20`
- Traffic **from Sales into the router** enters `g0/0.20`
- ACL applied: `ip access-group 100 in` on `g0/0.20`

```cisco
access-list 100 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
access-list 100 permit ip any any
```

| Flow | Result |
| ---- | ------ |
| Sales → HR | Matches deny on the way **in** `g0/0.20` → blocked |
| HR → Sales | Enters via `g0/0.10`, not filtered by the Sales ACL → allowed |

**Draw the arrow before you pick `in` or `out`.** If you apply the same ACL `out` on the wrong interface, you will block the wrong direction (or nothing useful).

### Second example — Guest filter

Guest LAN on `g0/2`. Block Guest from Internet (`any` destination off-box), allow other internal policy as designed:

```cisco
interface g0/2
 ip access-group GUEST-FILTER in
```

Packets **from Guest enter** the router on `g0/2` → filter them **inbound** at the source.

---

## Part 7 — Implicit Deny and `permit any any`

Every IPv4 ACL ends with an invisible:

```text
deny ip any any
```

So if you only write denies, you also block **everything else** (including return traffic you still need).

**Typical filtering ACL pattern:**

1. Specific **deny** lines for unwanted flows
2. Broad **`permit ip any any`** (or tighter permits) for everything else you still want
3. Implicit deny remains, but often never reached

```cisco
access-list 100 deny tcp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 eq 23
access-list 100 permit ip any any
```

**Exam trap:** Forgetting `permit ip any any` after a deny → “ACL broke the whole network.”

---

## Part 8 — Protocol and Port Matching (Extended)

Extended ACL syntax (conceptual order):

```text
access-list <num> {permit|deny} <protocol> <src> <src-wildcard> [port] <dst> <dst-wildcard> [port]
```

Examples:

```cisco
! Block Telnet (TCP 23) from Sales to HR
access-list 110 deny tcp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 eq 23
access-list 110 permit ip any any

! Block all IP from Guest to anywhere (then permit others in a named list)
ip access-list extended GUEST-FILTER
 deny ip 192.168.99.0 0.0.0.255 any
 permit ip any any
```

| Keyword | Meaning |
| ------- | ------- |
| `eq 80` | Equal to port 80 |
| `gt` / `lt` | Greater / less than (less common in intro labs) |
| `est` / `established` | TCP established (legacy filtering idea — awareness) |
| `host x.x.x.x` | Single host (wildcard `0.0.0.0`) |
| `any` | All addresses |

Common ports to recognize: `22` SSH, `23` Telnet, `53` DNS, `80` HTTP, `443` HTTPS.

---

## Part 9 — Where ACLs Live vs Where NAT ACLs Live

| ACL purpose | Typical role |
| ----------- | ------------ |
| **Security filter** (`ip access-group`) | Permit/deny forwarding of packets |
| **NAT match list** (`ip nat inside source list`) | Select addresses **eligible to translate** |

Do not confuse them. A NAT ACL **permit** means “may be translated,” not “allowed through a firewall.”

Details: [IP-addresses/NAT_PAT_Notes.md](IP-addresses/NAT_PAT_Notes.md).

---

## Part 10 — Applying and Verifying

```cisco
interface g0/0.20
 ip access-group 100 in
!
show access-lists
show ip interface g0/0.20
```

| Command | Use |
| ------- | --- |
| `show access-lists` | See entries and **match hit counts** |
| `show ip interface <intf>` | Confirm which ACL is inbound/outbound |
| `show running-config \| include access` | Quick config hunt |

**Hit counts** are your best friend in Packet Tracer: generate the blocked flow and watch the deny line increment.

---

## Part 11 — Router-on-a-Stick Reminder

With ROAS, each VLAN is a **subinterface**. Applying an ACL to `g0/0.20 in` filters traffic **from that VLAN into the router** — perfect for inter-VLAN policy because the router is the only path between VLANs.

Related: [inter_vlan_routing.md](inter_vlan_routing.md) · [labs/ACL_Router-on-a-stick_VLAN/CCNA_ACL_Lab.md](labs/ACL_Router-on-a-stick_VLAN/CCNA_ACL_Lab.md)

---

## Part 12 — Common Mistakes & Exam Traps

| Wrong idea | Correct idea |
| ---------- | ------------ |
| “ACLs are checked bottom-up” | **Top-down**; first match wins |
| “No permit needed if I only deny bad traffic” | **Implicit deny** kills the rest unless you permit |
| “`in` means into the LAN” | `in` means **into the router** on that interface |
| “Standard ACL can match destination ports” | Standard = **source IP only** |
| “Wildcard `255.255.255.0` means /24” | That wildcard is almost always **wrong** for /24; use `0.0.0.255` |
| “Named and numbered are different features” | Same matching power; named is clearer to manage |
| “One ACL direction filters both ways automatically” | ACLs are **directional**; return traffic is a separate path |

---

## Part 13 — Practice Questions (Self-Check)

1. What happens if a packet matches no ACL line?
2. Standard ACLs match which header fields?
3. Extended ACL numbers fall in which ranges?
4. Should an extended ACL usually be placed near the source or destination?
5. In `ip access-group 100 in` on `g0/0.20`, when is the ACL evaluated?
6. Why do most filtering ACLs end with `permit ip any any`?
7. Convert mask `255.255.255.252` to a wildcard.

### Answers

1. It is **denied** by the **implicit deny**.
2. **Source IP** only.
3. **100–199** and **2000–2699**.
4. Near the **source** (drop unwanted traffic early, with precision).
5. When packets **enter the router** through `g0/0.20`.
6. Without it, the implicit deny blocks **all other** traffic too.
7. **`0.0.0.3`**.

---

## Part 14 — Quick Reference Card

```text
ACL = ordered permit/deny list; first match wins
Ends with invisible deny ip any any

Standard: source only (1–99, 1300–1999) — place near destination
Extended: src/dst/protocol/ports (100–199, 2000–2699) — place near source

Named ACLs: ip access-list {standard|extended} NAME
Apply: ip access-group {num|NAME} {in|out}

in  = into the router on that interface
out = out of the router on that interface

Wildcard 0 bits = match, 1 bits = ignore
/24 → 0.0.0.255    host → 0.0.0.0 or "host"

Verify: show access-lists (watch hit counts)
```

---

**Mastery check:** Rebuild the Sales→HR deny from the [small office lab](labs/CCNA_Small_Office_Network_Lab.md) from memory, predict which ping fails, then confirm with `show access-lists` hit counts — direction + first-match + implicit deny all in one test.
