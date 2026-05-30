# Subnetting — Extensive Study Guide

**Goal:** Read a network requirement, **design subnets**, and list **network address**, **broadcast**, **usable host range**, and **gateway** without guessing. This guide builds from binary basics through **VLSM** (Variable Length Subnet Masking) with full worked examples.

---

## Part 1 — Core Vocabulary


| Term                  | Meaning                                                                                        |
| --------------------- | ---------------------------------------------------------------------------------------------- |
| **IPv4 address**      | 32 bits, usually written as four decimal octets (0–255), e.g. `192.168.10.45`                  |
| **Subnet mask**       | Tells which bits are **network** vs **host**; often written in **CIDR** form                   |
| **CIDR notation**     | `192.168.10.0/24` — slash number = **prefix length** = count of **leading 1-bits** in the mask |
| **Network address**   | First address in subnet; **host bits all 0** — identifies the subnet itself                    |
| **Broadcast address** | Last address in subnet; **host bits all 1** — sent to every host on the subnet                 |
| **Usable hosts**      | Addresses between network and broadcast (typically **cannot** assign those two to hosts)       |
| **Default gateway**   | Usually first **usable** host IP on a subnet (convention, not a rule)                          |
| **VLSM**              | Using **different prefix lengths** in the same address space to minimize waste                 |


---

## Part 2 — The One Formula You Must Know

For IPv4 subnet with prefix length **p**:


| Item                                | Formula        |
| ----------------------------------- | -------------- |
| Total addresses in subnet           | 2^{(32-p)}     |
| Usable host addresses (typical LAN) | 2^{(32-p)} - 2 |
| Host bits                           | 32 - p         |


**Why minus 2?** Network address (all host bits 0) and broadcast (all host bits 1) are reserved on most LAN subnets.

**Exceptions (know for exams):** `/31` can be used for point-to-point links (RFC 3021); `/32` is a single host route.

---

## Part 3 — Quick Reference — Common Prefixes


| CIDR | Subnet mask                          | Host bits | Total addrs | Usable hosts   |
| ---- | ------------------------------------ | --------- | ----------- | -------------- |
| /32  | 255.255.255.255                      | 0         | 1           | 1 (host route) |
| /30  | 255.255.255.252                      | 2         | 4           | 2 (often P2P)  |
| /29  | 255.255.255.248                      | 3         | 8           | 6              |
| /28  | 255.255.255.240                      | 4         | 16          | 14             |
| /27  | 255.255.255.224                      | 5         | 32          | 30             |
| /26  | 255.255.255.192                      | 6         | 64          | 62             |
| /25  | 255.255.255.128                      | 7         | 128         | 126            |
| /24  | 255.255.255.0                        | 8         | 256         | 254            |
| /23  | 255.255.255.0 → actually 255.254.0.0 | 9         | 512         | 510            |
| /22  | 255.255.252.0                        | 10        | 1024        | 1022           |
| /21  | 255.255.248.0                        | 11        | 2048        | 2046           |
| /20  | 255.255.240.0                        | 12        | 4096        | 4094           |
| /16  | 255.255.0.0                          | 16        | 65,536      | 65,534         |
| /8   | 255.0.0.0                            | 24        | 16,777,216  | 16,777,214     |


---

## Part 4 — Binary Basics (Minimum You Need)

Each octet = **8 bits**. Decimal 255 = `11111111`; decimal 0 = `00000000`.

### 4.1 Powers of Two (memorize through 2^16)


| n   | 2^n   |
| --- | ----- |
| 8   | 256   |
| 9   | 512   |
| 10  | 1024  |
| 11  | 2048  |
| 12  | 4096  |
| 13  | 8192  |
| 14  | 16384 |
| 15  | 32768 |
| 16  | 65536 |


### 4.2 Convert One Octet — Example

**192** in binary:

```
128 + 64 = 192  →  11000000
```

**224** in binary (common in /27 masks):

```
128 + 64 + 32 = 224  →  11100000
```

### 4.3 Mask ↔ CIDR

`/26` on a classless network means **26 network bits**:

```
255.255.255.192
|←—— 24 fixed ——→|← 2 host bits in last octet →|
11111111.11111111.11111111.11000000
```

---

## Part 5 — Method: Find Network, Broadcast, and Host Range

Use this **four-step method** on any subnet.

### Step 1 — Identify prefix length and host bits

Example: `10.50.120.75/26`  

- Host bits = 32 − 26 = **6**  
- Block size in last octet = 2^6 = **64** (when subnet boundary falls in the 4th octet)

### Step 2 — Find the block (subnet) boundary

Divide the **interesting octet** by block size; find which block the IP falls into.

For `10.50.120.75/26`, block size in 4th octet = 64:

```
0, 64, 128, 192 ...
75 is in the 64–127 block  →  network 4th octet = 64
```

### Step 3 — Network and broadcast

- **Network:** `10.50.120.64/26`  
- **Broadcast:** last address in block = `10.50.120.127` (64 + 64 − 1)

### Step 4 — Usable hosts

- **First usable:** `10.50.120.65`  
- **Last usable:** `10.50.120.126`

### Summary table


| Field        | Value             |
| ------------ | ----------------- |
| Given        | `10.50.120.75/26` |
| Network      | `10.50.120.64/26` |
| First host   | `10.50.120.65`    |
| Last host    | `10.50.120.126`   |
| Broadcast    | `10.50.120.127`   |
| Usable count | 62                |


---

## Part 6 — Creating Subnets (Subnetting Down)

**Subnetting down** = take a **larger** block and split into **smaller** subnets (longer prefix = more network bits).

### Rule of thumb

If you borrow **n** bits from host portion into network portion:

- Number of new subnets = 2^n  
- New prefix length = old prefix + n  
- Hosts per new subnet = 2^{(\text{remaining host bits})} - 2

---

## Example 1 — Split a /24 into Four Equal Subnets

**Given:** `192.168.1.0/24`  
**Need:** 4 subnets (e.g. Sales, Engineering, Guest, Servers)

### Step 1 — How many bits to borrow?

Need 4 subnets → 2^n \ge 4 → borrow **n = 2** bits.

### Step 2 — New prefix

`/24 + 2 = /26`

Each subnet has 2^{32-26} = 64 addresses → **62 usable hosts** each.

### Step 3 — Block size in 4th octet

2^{(32-26)} in last octet when /26 → block size **64** in 4th octet: 0, 64, 128, 192.

### Step 4 — List all subnets


| Subnet # | Network            | Usable range    | Broadcast       |
| -------- | ------------------ | --------------- | --------------- |
| 1        | `192.168.1.0/26`   | `.1` – `.62`    | `192.168.1.63`  |
| 2        | `192.168.1.64/26`  | `.65` – `.126`  | `192.168.1.127` |
| 3        | `192.168.1.128/26` | `.129` – `.190` | `192.168.1.191` |
| 4        | `192.168.1.192/26` | `.193` – `.254` | `192.168.1.255` |


### Step 5 — Assign names (design doc style)


| VLAN | Department  | Network            | Gateway (typical) |
| ---- | ----------- | ------------------ | ----------------- |
| 10   | Sales       | `192.168.1.0/26`   | `192.168.1.1`     |
| 20   | Engineering | `192.168.1.64/26`  | `192.168.1.65`    |
| 30   | Guest       | `192.168.1.128/26` | `192.168.1.129`   |
| 40   | Servers     | `192.168.1.192/26` | `192.168.1.193`   |


**Check:** Four subnets, no overlap, all fit inside original `/24`.

---

## Example 2 — Split a /24 into Eight /27 Subnets

**Given:** `172.16.5.0/24`  
**Need:** 8 equal subnets

Borrow **3** bits → `/27`  
Block size in 4th octet = 2^{(32-27)} = 32


| #   | Network           | Broadcast      |
| --- | ----------------- | -------------- |
| 1   | `172.16.5.0/27`   | `172.16.5.31`  |
| 2   | `172.16.5.32/27`  | `172.16.5.63`  |
| 3   | `172.16.5.64/27`  | `172.16.5.95`  |
| 4   | `172.16.5.96/27`  | `172.16.5.127` |
| 5   | `172.16.5.128/27` | `172.16.5.159` |
| 6   | `172.16.5.160/27` | `172.16.5.191` |
| 7   | `172.16.5.192/27` | `172.16.5.223` |
| 8   | `172.16.5.224/27` | `172.16.5.255` |


Each /27 → **30 usable hosts**.

---

## Example 3 — Choose the Right Prefix for Host Count

**Question:** What is the **smallest** subnet (longest prefix) that fits **50 hosts**?

Need usable ≥ 50 → total addresses ≥ 52 (with network + broadcast).


| Prefix | Usable hosts | Fits 50? |
| ------ | ------------ | -------- |
| /26    | 62           | Yes      |
| /27    | 30           | No       |


**Answer:** `/26` per subnet.

If given `10.10.0.0/24` and one LAN needs 50 hosts:

- Use one `/26` → e.g. `10.10.0.0/26` (`.1`–`.62`)  
- Remaining space for other subnets starts at `10.10.0.64` (next /26 boundary).

---

## Part 7 — VLSM (Variable Length Subnet Masking)

**VLSM** = carve one block into subnets of **different sizes** by assigning the **largest requirement first**, then continuing sequentially without overlap.

### VLSM workflow (always follow this order)

1. **Sort** subnet requirements by **largest host count first**.
2. For each requirement, pick the **smallest prefix** that fits (power-of-two host space).
3. Assign the **next available** network address aligned to that prefix’s block size.
4. **Never** reuse space; next subnet starts after previous subnet’s broadcast + 1 (i.e. next aligned block).

---

## Example 4 — Full VLSM from One /24

**Given block:** `192.168.100.0/24`  

**Requirements:**


| Site / LAN         | Hosts needed |
| ------------------ | ------------ |
| HQ LAN             | 100          |
| Branch LAN         | 50           |
| Guest Wi‑Fi        | 25           |
| Point-to-point WAN | 2            |
| Management         | 5            |


### Step 1 — Sort largest → smallest

100 → 50 → 25 → 5 → 2

### Step 2 — Pick prefix per requirement


| Hosts needed | Min usable | Prefix | Usable provided |
| ------------ | ---------- | ------ | --------------- |
| 100          | 100        | /25    | 126             |
| 50           | 50         | /26    | 62              |
| 25           | 25         | /27    | 30              |
| 5            | 5          | /29    | 6               |
| 2            | 2          | /30    | 2               |


### Step 3 — Assign sequentially

**1) HQ — 100 hosts → /25**

- Network: `192.168.100.0/25`  
- Range: `.1` – `.126`  
- Broadcast: `192.168.100.127`  
- Next free address: `192.168.100.128`

**2) Branch — 50 hosts → /26**

- Network: `192.168.100.128/26`  
- Range: `.129` – `.190`  
- Broadcast: `192.168.100.191`  
- Next free: `192.168.100.192`

**3) Guest — 25 hosts → /27**

- Network: `192.168.100.192/27`  
- Range: `.193` – `.222`  
- Broadcast: `192.168.100.223`  
- Next free: `192.168.100.224`

**4) Management — 5 hosts → /29**

- Network: `192.168.100.224/29`  
- Range: `.225` – `.230`  
- Broadcast: `192.168.100.231`  
- Next free: `192.168.100.232`

**5) WAN P2P — 2 hosts → /30**

- Network: `192.168.100.232/30`  
- Range: `.233` – `.234` (typical router endpoints)  
- Broadcast: `192.168.100.235`  
- Next free: `192.168.100.236`

### Step 4 — Final design table


| Name   | Network              | Mask            | Gateway                | Usable range  |
| ------ | -------------------- | --------------- | ---------------------- | ------------- |
| HQ     | `192.168.100.0/25`   | 255.255.255.128 | `192.168.100.1`        | `.1`–`.126`   |
| Branch | `192.168.100.128/26` | 255.255.255.192 | `192.168.100.129`      | `.129`–`.190` |
| Guest  | `192.168.100.192/27` | 255.255.255.224 | `192.168.100.193`      | `.193`–`.222` |
| Mgmt   | `192.168.100.224/29` | 255.255.255.248 | `192.168.100.225`      | `.225`–`.230` |
| WAN    | `192.168.100.232/30` | 255.255.255.252 | R1: `.233`, R2: `.234` | `.233`–`.234` |


**Remaining space:** `192.168.100.236` – `192.168.100.255` (20 addresses) — spare for future /29 or /30.

---

## Example 5 — VLSM from a /22 Corporate Block

**Given:** `10.20.0.0/22` (1024 addresses, 1022 usable if treated as one LAN — you will subdivide)

**Requirements:**


| LAN            | Hosts  |
| -------------- | ------ |
| Data center    | 400    |
| Office floor 1 | 120    |
| Office floor 2 | 60     |
| VoIP           | 30     |
| Two WAN links  | 2 each |


### Prefix selection


| Hosts | Prefix | Block size (addresses) |
| ----- | ------ | ---------------------- |
| 400   | /23    | 512                    |
| 120   | /25    | 128                    |
| 60    | /26    | 64                     |
| 30    | /27    | 32                     |
| 2     | /30    | 4                      |


### Assignments (watch boundaries in **3rd octet** for /22)

**1) Data center /23** — spans two /24s inside /22  

- Network: `10.20.0.0/23`  
- Broadcast: `10.20.1.255`  
- Next start: `10.20.2.0`

**2) Floor 1 /25**

- Network: `10.20.2.0/25`  
- Broadcast: `10.20.2.127`  
- Next: `10.20.2.128`

**3) Floor 2 /26**

- Network: `10.20.2.128/26`  
- Broadcast: `10.20.2.191`  
- Next: `10.20.2.192`

**4) VoIP /27**

- Network: `10.20.2.192/27`  
- Broadcast: `10.20.2.223`  
- Next: `10.20.2.224`

**5) WAN1 /30**

- Network: `10.20.2.224/30`  
- Broadcast: `10.20.2.227`  
- Next: `10.20.2.228`

**6) WAN2 /30**

- Network: `10.20.2.228/30`  
- Broadcast: `10.20.2.231`  
- Next: `10.20.2.232`

**Remaining** within `/22`: `10.20.2.232` – `10.20.3.255` for growth.

---

## Part 8 — Subnets That Cross Octet Boundaries (/23, /22, etc.)

When prefix is **less than /24**, block size affects **third** (or second) octet.

### Example — /23 boundaries

`/23` = 512 addresses = two consecutive /24s.

Networks align on **even** third octets:

```
10.1.0.0/23   → 10.1.0.0 – 10.1.1.255
10.1.2.0/23   → 10.1.2.0 – 10.1.3.255
10.1.4.0/23   → 10.1.4.0 – 10.1.5.255
```

**Wrong:** `10.1.1.0/23` — not aligned (third octet must be even for standard /23 split from /16-style planning).

### Block size cheat (where the boundary lives)


| Prefix | Block size (addresses) | Often boundary in       |
| ------ | ---------------------- | ----------------------- |
| /25    | 128                    | 4th octet               |
| /24    | 256                    | 4th octet               |
| /23    | 512                    | 3rd octet (steps of 2)  |
| /22    | 1024                   | 3rd octet (steps of 4)  |
| /21    | 2048                   | 3rd octet (steps of 8)  |
| /20    | 4096                   | 3rd octet (steps of 16) |
| /16    | 65536                  | 2nd octet               |


---

## Example 6 — “Which Subnet Is This Host In?”

**Host:** `172.16.40.200/27`

Block size = 32 in 4th octet. Multiples of 32: 0, 32, 64, 96, 128, 160, 192, 224.

200 falls in **192–223** block.


| Answer    | Value                             |
| --------- | --------------------------------- |
| Network   | `172.16.40.192/27`                |
| Broadcast | `172.16.40.223`                   |
| Usable    | `172.16.40.193` – `172.16.40.222` |


Is `.200` usable? **Yes.**

---

## Example 7 — Invalid Addresses (Exam Traps)

For subnet `192.168.50.64/26`:


| Address          | Valid host? | Why                                   |
| ---------------- | ----------- | ------------------------------------- |
| `192.168.50.64`  | No          | Network address                       |
| `192.168.50.65`  | Yes         | First usable                          |
| `192.168.50.127` | No          | Broadcast                             |
| `192.168.50.128` | No          | Belongs to **next** subnet (`128/26`) |


---

## Part 9 — Route Summarization (Supernetting)

**Summarization** = one route advertises **many contiguous subnets** — opposite direction from VLSM (combine, not split).

### Rules (simplified)

1. Subnets must be **contiguous**.
2. Summary prefix covers all of them with **same network bits** in common.
3. Summary network address = **lowest** subnet, mask = **shortest** prefix that still excludes outsiders.

### Example — Summarize these routes

```
192.168.0.0/26
192.168.0.64/26
192.168.0.128/26
192.168.0.192/26
```

All four are `/26` inside `192.168.0.0/24` — they **exactly fill** the /24.

**Summary:** `192.168.0.0/24`

### Example — Partial summary

```
10.10.0.0/24
10.10.1.0/24
10.10.2.0/24
10.10.3.0/24
```

Four /24s aligned → **summary `10.10.0.0/22`**

Binary view: third octet varies 0–3 in last **2** bits of third octet → borrow from /16 to /22.

---

## Part 10 — Private Address Space (Design Starting Points)


| RFC 1918 block   | Default feel                             |
| ---------------- | ---------------------------------------- |
| `10.0.0.0/8`     | Huge enterprises / labs                  |
| `172.16.0.0/12`  | Medium (`172.16.0.0` – `172.31.255.255`) |
| `192.168.0.0/16` | Small office / home lab                  |


**Lab tip:** Pick one block per topology (e.g. `10.50.0.0/16`) and VLSM inside it so routes stay clearly private.

---

## Part 11 — Practice Problems (Try First, Then Check)

### Problem A

Given `203.0.113.0/27`, list network, first/last host, broadcast, and usable count.

Solution

Block size 32 → only one subnet in this allocation.


|            |                  |
| ---------- | ---------------- |
| Network    | `203.0.113.0/27` |
| First host | `203.0.113.1`    |
| Last host  | `203.0.113.30`   |
| Broadcast  | `203.0.113.31`   |
| Usable     | 30               |




### Problem B

Split `10.1.1.0/24` into **2** equal subnets. Write both networks in CIDR.

Solution

Borrow 1 bit → `/25`.

1. `10.1.1.0/25` (`.1`–`.126`, broadcast `.127`)
2. `10.1.1.128/25` (`.129`–`.254`, broadcast `.255`)



### Problem C — VLSM

From `172.16.10.0/24`, allocate for: **60** hosts, **28** hosts, **12** hosts, **2** hosts (WAN). Write full table.

Solution

Order: 60 → 28 → 12 → 2  
Prefixes: /26, /27, /28, /30  


| LAN      | Network            |
| -------- | ------------------ |
| 60 hosts | `172.16.10.0/26`   |
| 28 hosts | `172.16.10.64/27`  |
| 12 hosts | `172.16.10.96/28`  |
| WAN      | `172.16.10.112/30` |


Remaining: `172.16.10.116` – `172.16.10.255`



---

## Part 12 — Speed Tricks for Exams and Labs

### 12.1 Last octet block sizes (for /25 through /32)


| Prefix | Increment in 4th octet |
| ------ | ---------------------- |
| /25    | 128                    |
| /26    | 64                     |
| /27    | 32                     |
| /28    | 16                     |
| /29    | 8                      |
| /30    | 4                      |


**Trick:** To find network of `x` in last octet with increment **I**:


\text{networkoctet} = \lfloor x / I \rfloor \times I


Example: `x=75`, `/26` → I=64 → floor(75/64)×64 = **64**.

### 12.2 “Magic number” for hosts needed

Find smallest **2^h − 2 ≥ required hosts** → h host bits → prefix = **32 − h**.

---

## Part 13 — Document Your Designs (Professional Habit)

When you subnet for Packet Tracer or work, always output:

```
| VLAN | Name   | Network          | Mask            | Gateway        | DHCP range        |
|------|--------|------------------|-----------------|----------------|-------------------|
| 10   | Users  | 192.168.100.0/25 | 255.255.255.128 | 192.168.100.1  | .10 – .200        |
```

Include **DNS**, **DHCP excluded** gateway/broadcast, and **reserved** ranges for infrastructure.

---

## Part 14 — Common Mistakes

1. **Overlapping subnets** — usually from wrong VLSM order or misaligned block.
2. **Using network/broadcast as host IPs** — breaks routing and DHCP.
3. **Wrong mask on a host** — host thinks neighbor is remote → sends to wrong gateway.
4. **Summarizing non-contiguous ranges** — blackholes traffic.
5. **Forgetting /30 and /31** on WAN links when doing VLSM — allocate **before** tiny LANs if policy says WAN is fixed, or follow “largest first” consistently.

---

## Part 15 — IPv6 Prefix Basics (Brief)

IPv6 uses **128-bit** addresses; subnetting is still **prefix length**, but space is huge so you usually assign `**/64`** per LAN (standard SLAAC segment).

Example: `2001:db8:acad::/48` → subnets like:

```
2001:db8:acad:1::/64
2001:db8:acad:2::/64
2001:db8:acad:3::/64
```

No broadcast; no “minus 2” host math on LAN `/64`s for human planning. Focus IPv4 VLSM first; apply the **same alignment discipline** to hex boundaries when you advance.

---

## Quick Workflow Card (Print This)

```
1. Count hosts needed → pick prefix (/26, /27, …)
2. List block size = 2^(host bits)
3. Align network to block boundary
4. Network = first address; Broadcast = last; Hosts = middle
5. VLSM: largest first, next subnet starts after broadcast block
6. Verify: no overlap, all inside parent block
```

---

**Mastery check:** Given a random `/24` and four department sizes, produce a full VLSM table in **under 10 minutes** with zero overlaps. Repeat until boring — that’s when subnetting stops slowing you down in CCNA labs and interviews.