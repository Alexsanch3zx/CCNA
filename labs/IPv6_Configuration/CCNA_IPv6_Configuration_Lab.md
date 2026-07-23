# CCNA Packet Tracer Lab – IPv6 Addressing & Static Routing

## Lab Overview

Your IPv4 labs already cover subnets, gateways, and static routes. This lab rebuilds that same story in **IPv6** — same skills, different address language.

You will:

1. Build a **two-site** topology (LAN A ↔ R1 ↔ R2 ↔ LAN B)
2. Enable **IPv6 unicast routing** and assign **global unicast** addresses
3. Confirm **link-local** addresses appear automatically (NDP / neighbor discovery)
4. Configure PCs with static IPv6, then try **SLAAC** on one LAN
5. Add **IPv6 static routes** so Site A reaches Site B
6. Verify with `ping`, `show ipv6 route`, and `show ipv6 neighbors`

**Level:** Intermediate — explanations stay beginner-friendly; the topology and routing steps match a multi-site IPv4 lab.

**Prerequisite:** Basic router CLI, interface addressing, and static routes from earlier labs. Skim [IPv4 vs IPv6](../../IP-addresses/IPv4_vs_IPv6_Study_Guide.md) first.

**Cheat sheet:** [IPv6 Essentials](../../cheat-sheets/cisco-networking-command-cheat-sheet.md#ipv6-essentials)

**Why this is new:** You have IPv6 terms and commands in notes, but no dedicated hands-on lab. This is the first full IPv6 Packet Tracer walkthrough in the repo.

---

# Key Concepts (Quick Review)


| Term                 | Meaning                                                                 |
| -------------------- | ----------------------------------------------------------------------- |
| **GUA**              | Global Unicast Address — routable (`2000::/3`; labs use `2001:db8::/32`) |
| **LLA**              | Link-Local Address — starts with `fe80::/10`; auto on every IPv6 interface |
| **ULA**              | Unique Local Address — `fc00::/7` (private-ish; optional stretch)       |
| **NDP**              | Neighbor Discovery Protocol — replaces ARP (NS/NA, RS/RA)               |
| **SLAAC**            | Stateless Address Autoconfiguration — host builds address from RA prefix |
| **`ipv6 unicast-routing`** | Turns the router into an IPv6 **router** (required for forwarding + RAs) |


**Mental model:**

- Every IPv6 interface gets a **link-local** address even if you never type one.
- You still assign a **GUA** (`2001:db8:…`) for real end-to-end connectivity across routers.
- Cross-site reachability still needs a **route** — just like IPv4 static routes, but with `ipv6 route`.

**Exam-friendly facts:**

- IPv6 has **no broadcast** — multicast + anycast instead.
- `/64` is the normal LAN prefix length.
- Documentation/lab prefix: **`2001:db8::/32`** (safe to use in examples).

---

# Network Topology

```text
  Site A (LAN)                         Site B (LAN)
  2001:db8:1::/64                      2001:db8:2::/64
        |                                    |
      PC0                                  PC1
        |                                    |
      SW1                                  SW2
        |                                    |
   R1 G0/0 ::1                          R2 G0/0 ::1
        |                                    |
   R1 G0/1 ::1 ----- 2001:db8:12::/64 ----- R2 G0/1 ::2
              (serial or Ethernet link)
```

**Story:** Two branch sites. Each site has one PC on a `/64` LAN. R1 and R2 share a point-to-point link. After you configure addresses, PCs can ping their own gateway. After you add static routes, PC0 can ping PC1 over IPv6.

Topology diagram: [CCNA_IPv6_Configuration_Topology.puml](CCNA_IPv6_Configuration_Topology.puml) (same folder).

---

# Devices Needed


| Device                   | Role                                      |
| ------------------------ | ----------------------------------------- |
| 1× Router (2911) – `R1`  | Site A gateway + static routes            |
| 1× Router (2911) – `R2`  | Site B gateway + static routes            |
| 1× Switch (2960) – `SW1` | Site A access (Layer 2 only)              |
| 1× Switch (2960) – `SW2` | Site B access (Layer 2 only)              |
| 1× PC – `PC0`            | Site A host                               |
| 1× PC – `PC1`            | Site B host                               |


**Cabling (copper straight-through unless noted):**

| From | Port   | To  | Port   |
| ---- | ------ | --- | ------ |
| R1   | Gig0/0 | SW1 | Fa0/24 |
| PC0  | Fa0    | SW1 | Fa0/1  |
| R2   | Gig0/0 | SW2 | Fa0/24 |
| PC1  | Fa0    | SW2 | Fa0/1  |
| R1   | Gig0/1 | R2  | Gig0/1 |

> If Gig0/1–Gig0/1 stays down in your PT version, use a **Serial** cable (e.g. R1 Se0/0/0 ↔ R2 Se0/0/0) and put the WAN addresses on those serial interfaces instead. Commands below stay the same — only the interface names change.

---

# Address Plan


| Segment     | Prefix             | Device / Interface | Address            |
| ----------- | ------------------ | ------------------ | ------------------ |
| Site A LAN  | `2001:db8:1::/64`  | R1 Gig0/0          | `2001:db8:1::1/64` |
| Site A LAN  | `2001:db8:1::/64`  | PC0                | `2001:db8:1::10/64` |
| Site B LAN  | `2001:db8:2::/64`  | R2 Gig0/0          | `2001:db8:2::1/64` |
| Site B LAN  | `2001:db8:2::/64`  | PC1                | `2001:db8:2::10/64` |
| R1 ↔ R2 WAN | `2001:db8:12::/64` | R1 Gig0/1          | `2001:db8:12::1/64` |
| R1 ↔ R2 WAN | `2001:db8:12::/64` | R2 Gig0/1          | `2001:db8:12::2/64` |


| Host | Default gateway (IPv6) |
| ---- | ---------------------- |
| PC0  | `2001:db8:1::1`        |
| PC1  | `2001:db8:2::1`        |


**Route plan (after local connectivity works):**

| Router | Destination        | Next hop           |
| ------ | ------------------ | ------------------ |
| R1     | `2001:db8:2::/64`  | `2001:db8:12::2`   |
| R2     | `2001:db8:1::/64`  | `2001:db8:12::1`   |

---

# Step 0 — Build and Cable

1. Drag devices into Packet Tracer and rename them (`R1`, `R2`, `SW1`, `SW2`, `PC0`, `PC1`).
2. Cable per the table above.
3. Confirm link lights are green before configuring.

**Do steps in order.** After each router CLI block, run `write memory`. Save your `.pkt` when done.

---

# Step 1 — Enable IPv6 Routing and Address R1

On **R1**:

```cisco
enable
configure terminal
hostname R1

ipv6 unicast-routing

interface GigabitEthernet0/0
 description Site-A-LAN
 ipv6 address 2001:db8:1::1/64
 no shutdown
exit

interface GigabitEthernet0/1
 description WAN-to-R2
 ipv6 address 2001:db8:12::1/64
 no shutdown
exit

end
write memory
```

**Check:**

```cisco
show ipv6 interface brief
```

**Look for:**

- `GigabitEthernet0/0` and `GigabitEthernet0/1` **[up/up]**
- Each interface shows a **link-local** (`FE80::…`) **and** your GUA (`2001:DB8:…`)

Example shape (exact FE80 values differ — EUI-64 / MAC based):

```text
R1# show ipv6 interface brief
GigabitEthernet0/0     [up/up]
    FE80::… 
    2001:DB8:1::1
GigabitEthernet0/1     [up/up]
    FE80::…
    2001:DB8:12::1
```

**Why `ipv6 unicast-routing` matters:** Without it, the device can have IPv6 addresses but will **not** forward packets between interfaces (and will not send Router Advertisements for SLAAC later).

---

# Step 2 — Enable IPv6 Routing and Address R2

On **R2**:

```cisco
enable
configure terminal
hostname R2

ipv6 unicast-routing

interface GigabitEthernet0/0
 description Site-B-LAN
 ipv6 address 2001:db8:2::1/64
 no shutdown
exit

interface GigabitEthernet0/1
 description WAN-to-R1
 ipv6 address 2001:db8:12::2/64
 no shutdown
exit

end
write memory
```

**Check:**

```cisco
show ipv6 interface brief
ping 2001:db8:12::1
```

R2 should ping R1 across the WAN. If that fails, fix cabling / interface names / addresses before touching PCs.

---

# Step 3 — Switches (Access Ports Only)

Switches stay **Layer 2** — no IPv6 addressing required for this lab.

### SW1

```cisco
enable
configure terminal
hostname SW1

interface FastEthernet0/1
 switchport mode access
 switchport access vlan 1
 no shutdown
exit

interface FastEthernet0/24
 switchport mode access
 switchport access vlan 1
 no shutdown
exit

end
write memory
```

### SW2

```cisco
enable
configure terminal
hostname SW2

interface FastEthernet0/1
 switchport mode access
 switchport access vlan 1
 no shutdown
exit

interface FastEthernet0/24
 switchport mode access
 switchport access vlan 1
 no shutdown
exit

end
write memory
```

**Check:** Link lights green PC ↔ SW ↔ R. Default VLAN 1 is fine — this lab is about IPv6, not VLANs.

---

# Step 4 — Configure End Devices (Static IPv6)

In Packet Tracer, open each PC → **Desktop** → **IP Configuration**.

### PC0 (Site A)

| Field            | Value             |
| ---------------- | ----------------- |
| IPv6 Address     | `2001:db8:1::10`  |
| IPv6 Prefix Length | `64`            |
| IPv6 Gateway     | `2001:db8:1::1`   |

Leave IPv4 blank / unused for now (or ignore it — this lab is IPv6-first).

### PC1 (Site B)

| Field            | Value             |
| ---------------- | ----------------- |
| IPv6 Address     | `2001:db8:2::10`  |
| IPv6 Prefix Length | `64`            |
| IPv6 Gateway     | `2001:db8:2::1`   |

**Same-subnet checks first:**

From **PC0** Command Prompt:

```text
ping 2001:db8:1::1
```

From **PC1** Command Prompt:

```text
ping 2001:db8:2::1
```

Both should succeed. If not, verify PC gateway, prefix length `/64`, and that the router LAN interface is up.

**Cross-site ping now should fail** (no route yet) — that is expected:

```text
ping 2001:db8:2::10
```

---

# Step 5 — IPv6 Static Routes (Make Sites Reach Each Other)

### On R1 — how to reach Site B

```cisco
enable
configure terminal
ipv6 route 2001:db8:2::/64 2001:db8:12::2
end
write memory
```

### On R2 — how to reach Site A

```cisco
enable
configure terminal
ipv6 route 2001:db8:1::/64 2001:db8:12::1
end
write memory
```

**Check on both routers:**

```cisco
show ipv6 route
```

**Look for:**

- Connected routes for your local `/64`s (`C`)
- Static routes (`S`) for the remote LAN

Example shape on R1:

```text
S   2001:DB8:2::/64 [1/0]
     via 2001:DB8:12::2
C   2001:DB8:1::/64 …
C   2001:DB8:12::/64 …
```

**End-to-end tests:**

From **PC0**:

```text
ping 2001:db8:2::1
ping 2001:db8:2::10
```

From **PC1**:

```text
ping 2001:db8:1::1
ping 2001:db8:1::10
```

All four should succeed once both static routes exist.

---

# Step 6 — Neighbor Discovery (NDP) Reality Check

IPv6 does **not** use ARP. When a host talks on-link, it uses **Neighbor Solicitation / Neighbor Advertisement**.

On **R1** after PC0 has pinged the gateway:

```cisco
show ipv6 neighbors
```

**Look for:** PC0’s address `2001:DB8:1::10` mapped to a MAC on Gig0/0.

Also inspect an interface in detail:

```cisco
show ipv6 interface GigabitEthernet0/0
```

**Look for:**

- Global unicast address
- Link-local address
- Joined multicast groups (e.g. solicited-node)
- That the interface is advertising as a router (once `ipv6 unicast-routing` is on)

Optional Simulation mode tip: filter **ICMPv6** / NDP and watch NS/NA on a first ping to a new neighbor.

---

# Step 7 — Link-Local Ping (Optional but Useful)

You can ping using a **link-local** address, but you must specify the **outgoing interface** (link-locals are not unique across the whole network).

On **R1**:

```cisco
show ipv6 interface brief
```

Copy R2’s `FE80::…` on Gig0/1, then:

```cisco
ping FE80::xxxx%GigabitEthernet0/1
```

Packet Tracer support for the `%interface` syntax varies by version. If it fails, still note the exam idea: **link-local = same link only**; GUAs are what you use for multi-hop paths.

---

# Step 8 — SLAAC Stretch (Site A Only)

Once static addressing works, try **Stateless Address Autoconfiguration** on PC0.

1. On **PC0** → IP Configuration → clear the static IPv6 / set to **Auto Config** (wording varies by PT version: Auto Config / SLAAC / obtain automatically).
2. Confirm **R1** still has `ipv6 unicast-routing` (required for Router Advertisements).
3. Wait a few seconds, then check PC0’s IPv6 address.

**Expected behavior:**

- PC0 gets a GUA in `2001:db8:1::/64` (often EUI-64 based — not `::10`)
- Gateway points at R1 (via RA)

From PC0, ping `2001:db8:1::1` and then `2001:db8:2::10` again.

> If Auto Config does nothing in your PT build, leave PC0 static — SLAAC support in Packet Tracer is uneven. Know the **concept** for the exam even if the GUI is stubborn.

**Optional router-side note:** Some images support `ipv6 nd` tweaks; for CCNA you mainly need: **RA comes from a router with a `/64` on the LAN + unicast-routing enabled**.

---

# Step 9 — Dual-Stack Stretch (Optional)

Add IPv4 alongside IPv6 on the same interfaces (common real-world pattern).

Example on **R1** Gig0/0:

```cisco
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 ipv6 address 2001:db8:1::1/64
```

Give PC0 an IPv4 address (`192.168.1.10/24`, GW `192.168.1.1`) **and** keep its IPv6 settings. Ping both stacks:

```text
ping 192.168.1.1
ping 2001:db8:1::1
```

You do **not** need full IPv4 routing across sites for this stretch — the point is seeing both address families on one interface.

---

# Verification Checklist


| Check                                      | Command / Action                         | Pass when                          |
| ------------------------------------------ | ---------------------------------------- | ---------------------------------- |
| Interfaces have GUA + LLA                  | `show ipv6 interface brief`              | Both addresses present, up/up      |
| WAN works                                  | R2: `ping 2001:db8:12::1`                | Success                            |
| Local gateway                              | PC0 → `ping 2001:db8:1::1`               | Success                            |
| Static routes installed                    | `show ipv6 route`                        | Remote `/64` shows as `S`          |
| Cross-site                                 | PC0 → `ping 2001:db8:2::10`              | Success                            |
| Neighbor cache                             | `show ipv6 neighbors`                    | PC MAC appears after ping          |


Full verification pass on each router:

```cisco
show ipv6 interface brief
show ipv6 route
show ipv6 neighbors
ping 2001:db8:1::10
ping 2001:db8:2::10
ping 2001:db8:12::1
ping 2001:db8:12::2
```

---

# Troubleshooting


| Symptom                         | Likely cause                                      | Fix                                              |
| ------------------------------- | ------------------------------------------------- | ------------------------------------------------ |
| No GUA on interface             | Forgot `ipv6 address …/64` or interface shutdown  | Configure address; `no shutdown`                 |
| Has address but cannot forward  | Missing `ipv6 unicast-routing`                    | Add it in global config                          |
| PC cannot ping gateway          | Wrong gateway / prefix length / cable             | Confirm `/64`, GW = router LAN GUA               |
| Local OK, remote fails          | Missing static route on one or both routers       | Add both `ipv6 route` statements                 |
| Route present, still fails      | Wrong next-hop (`.1` vs `.2`)                     | Next-hop must be the **other** router’s WAN GUA  |
| Ping to FE80 fails oddly        | Link-local needs outgoing interface               | Use GUA for multi-hop tests                      |
| SLAAC gives nothing             | PT Auto Config quirk / no RA                      | Fall back to static; verify unicast-routing      |


**Common mistake:** Configuring IPv6 addresses but forgetting **`ipv6 unicast-routing`** — interfaces look fine, remote networks never work, and SLAAC RAs never appear.

**Another classic:** Static route next-hop pointing at **your own** WAN address instead of the peer.

---

# What You Should Be Able to Explain After This Lab

1. Difference between **GUA** and **link-local**, and why both show up on `show ipv6 interface brief`.
2. Why **`ipv6 unicast-routing`** is required on Cisco routers for this lab.
3. How an IPv6 static route mirrors an IPv4 static route (`network/prefix` + next-hop).
4. Why NDP / `show ipv6 neighbors` replaces ARP thinking.
5. What SLAAC needs from the router (RA + `/64` prefix) vs a manually set host address.

---

# Reflection Prompt (Optional)

When you finish, add a journal entry in the repo README:

- What felt different from IPv4 addressing?
- Did link-local addresses confuse you at first? Why?
- Which verification command helped the most?

---

*Lab: Intermediate IPv6 configuration — dual-site static routing, NDP, and optional SLAAC/dual-stack.*
