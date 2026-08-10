# Spanning Tree (STP / RSTP / Rapid PVST+) — Deep Dive

**Audience:** CCNA students who need to understand **Layer 2 loop prevention**, root bridge election, port roles/states, and Cisco’s **per-VLAN** STP modes.

**Related files:** [terminology.md](terminology.md) · [MAC-addresses.md](MAC-addresses.md) · [inter_vlan_routing.md](inter_vlan_routing.md) · [cheat-sheets/command-cheat-sheet.md](cheat-sheets/command-cheat-sheet.md) · [labs/Layer3_Switching_Redundancy/CCNA_Layer3_Switching_Redundancy_Lab.md](labs/Layer3_Switching_Redundancy/CCNA_Layer3_Switching_Redundancy_Lab.md) · [labs/Small_Office_Network/CCNA_Small_Office_Network_Lab.md](labs/Small_Office_Network/CCNA_Small_Office_Network_Lab.md)

---

## Part 1 — What Problem Does STP Solve?

Ethernet switches **flood** unknown unicasts, broadcasts, and multicasts. If two (or more) switches are connected with **redundant links** and there is **no loop prevention**, those flooded frames circulate forever.

Result of an L2 loop:

- **Broadcast storm** — CPU and bandwidth collapse
- **MAC table instability** — same MAC learned on different ports (“flapping”)
- **Duplicate frames** — hosts receive the same frame more than once

**Spanning Tree Protocol (STP)** builds a **loop-free logical tree** out of a physically redundant topology by **blocking** some ports.

**CCNA takeaway:** Redundancy is good. Uncontrolled L2 loops are catastrophic. STP keeps both: physical redundancy + one active forwarding path.

---

## Part 2 — Core Vocabulary

| Term | Meaning |
| ---- | ------- |
| **Root bridge** | The “center” of the tree — all paths are measured toward this switch |
| **Bridge ID (BID)** | Priority + MAC address; lowest BID wins root election |
| **Path cost** | Cost of links toward the root (lower = better); based on link speed |
| **Root port (RP)** | On a non-root switch, the best port **toward** the root |
| **Designated port (DP)** | Best port **on a segment** for sending traffic **toward** that segment from the root’s side |
| **Blocking / Alternate** | Port that is up but **does not forward** user frames (loop prevention) |
| **BPDU** | Bridge Protocol Data Unit — STP control frames switches exchange |

---

## Part 3 — Root Bridge Election

Every switch starts assuming **it** is the root and sends BPDUs. Switches compare Bridge IDs.

### 3.1 Bridge ID

```text
Bridge ID = Priority (16 bits) + MAC address (48 bits)
```

| Field | Default / notes |
| ----- | --------------- |
| **Priority** | Default **32768**; configurable in steps of **4096** (Cisco) |
| **MAC** | Switch base MAC — tie-breaker when priorities match |

**Lowest Bridge ID = root.**

Cisco often uses **extended system ID** (priority + VLAN ID), so default priority for VLAN 10 looks like `32768 + 10 = 32778`.

### 3.2 Forcing root placement (ops best practice)

Place the root near **distribution/core**, not a random access switch.

```cisco
spanning-tree vlan 10 root primary
spanning-tree vlan 20 root secondary
```

Or set priority explicitly:

```cisco
spanning-tree vlan 10 priority 4096
```

---

## Part 4 — How Each Switch Picks Port Roles

After the root is known:

1. **Root ports** — each non-root switch picks **one** best path to the root (lowest path cost; ties broken by neighbor BID, then port ID).
2. **Designated ports** — on every link/segment, one end becomes designated (forwards toward that segment).
3. **Remaining ports** — **block** (classic STP) or become **alternate/backup** (RSTP).

### 4.1 Path cost (common CCNA values)

| Link speed | Classic STP cost | Long-path cost (common today) |
| ---------- | ---------------- | ----------------------------- |
| 10 Mbps | 100 | 2,000,000 |
| 100 Mbps | 19 | 200,000 |
| 1 Gbps | 4 | 20,000 |
| 10 Gbps | 2 | 2,000 |

Exact numbers can vary by platform/mode — exams care that **faster = lower cost**.

### 4.2 Simple two-switch example

```text
        SW1 (root) ---------------- SW2
             \                      /
              \---- (redundant) ---/
```

- Both switches elect SW1 as root (lower BID).
- On SW2, one uplink is **Root Port** (best path to SW1).
- The other uplink is **Alternate/Blocking** so the triangle does not loop.
- If the forwarding uplink fails, the blocked port can take over.

---

## Part 5 — Classic STP (802.1D) Port States

Classic STP uses slow timers and these states:

| State | Forwards user frames? | Learns MACs? | Purpose |
| ----- | --------------------- | ------------ | ------- |
| **Disabled** | No | No | Admin down / STP disabled on port |
| **Blocking** | No | No | Loop prevention; still receives BPDUs |
| **Listening** | No | No | Preparing to forward; builds topology view |
| **Learning** | No | Yes | Builds MAC table before forwarding |
| **Forwarding** | Yes | Yes | Normal operation |

Convergence with classic timers is often on the order of **~30–50 seconds** — too slow for modern LANs.

**Timers (awareness):** Hello (2s), Max Age (20s), Forward Delay (15s).

---

## Part 6 — RSTP (802.1w) — What Changed

**Rapid Spanning Tree (RSTP)** keeps the same **tree idea** but converges much faster using **proposal/agreement** handshakes instead of waiting on long timers.

### 6.1 RSTP port roles

| Role | Meaning |
| ---- | ------- |
| **Root** | Best path to root (same idea as 802.1D) |
| **Designated** | Forwards toward a segment |
| **Alternate** | Backup path to root (blocked) |
| **Backup** | Backup designated on a shared segment (rare in modern point-to-point Ethernet) |

### 6.2 RSTP port states (simplified for CCNA)

| State | Rough mapping |
| ----- | ------------- |
| **Discarding** | Combines disabled/blocking/listening behavior for user traffic |
| **Learning** | Learning MACs |
| **Forwarding** | Forwarding user frames |

### 6.3 Edge ports

Access ports connected to PCs should **not** wait for STP convergence:

```cisco
interface fa0/1
 spanning-tree portfast
 spanning-tree bpduguard enable
```

| Feature | Purpose |
| ------- | ------- |
| **PortFast** | Moves edge port to forwarding quickly (for end hosts) |
| **BPDU Guard** | If a BPDU arrives on an edge port → **err-disable** (protects against rogue switch) |
| **Root Guard** | Prevents a downstream switch from becoming root on that port |
| **Loop Guard** | Helps when BPDUs stop unexpectedly on a redundant link |

**Never put PortFast on links between switches.**

---

## Part 7 — PVST+ and Rapid PVST+ (Cisco)

Ethernet loops are per **broadcast domain**. With VLANs, Cisco runs STP **per VLAN**.

| Mode | Idea |
| ---- | ---- |
| **PVST+** | Per-VLAN Spanning Tree (Cisco) — one STP instance **per VLAN** |
| **Rapid PVST+** | RSTP mechanics **per VLAN** — common CCNA default goal |

```cisco
spanning-tree mode rapid-pvst
```

**Why per VLAN matters:** You can make **SW1 root for VLAN 10** and **SW2 root for VLAN 20**, so different VLANs prefer different uplinks (load sharing across redundant links).

```cisco
! On preferred root for VLAN 10
spanning-tree vlan 10 root primary
spanning-tree vlan 20 root secondary

! On the other distribution switch
spanning-tree vlan 20 root primary
spanning-tree vlan 10 root secondary
```

**MST (Multiple Spanning Tree)** maps many VLANs into fewer instances — awareness level for CCNA; deeper at CCNP.

---

## Part 8 — Reading `show spanning-tree`

```cisco
show spanning-tree
show spanning-tree vlan 10
show spanning-tree root
```

Look for:

1. **Which switch is Root** (Root ID / “This bridge is the root”)
2. **Your port roles** — Root, Desg, Altn/Blk
3. **Port states** — FWD vs BLK / discarding
4. **Protocol** — `ieee` / `rstp` / Rapid PVST indication depending on platform output

**Lab tip:** In [CCNA_Small_Office_Network_Lab.md](labs/Small_Office_Network/CCNA_Small_Office_Network_Lab.md) you mostly **read** STP. In [CCNA_Layer3_Switching_Redundancy_Lab.md](labs/Layer3_Switching_Redundancy/CCNA_Layer3_Switching_Redundancy_Lab.md) you practice **root placement**, PortFast, and BPDU Guard with redundant paths.

---

## Part 9 — Failure and Convergence Story

1. Forwarding uplink fails (cable pull, interface down).
2. Downstream switch loses BPDUs / detects failure.
3. Alternate port is unblocked and becomes the new root/designated path.
4. With **RSTP/Rapid PVST+**, this is often **sub-second to a few seconds** on point-to-point links — much better than classic STP.

**What you should lab:**

- Note which port is Alternate before failure
- Pull the active link
- Re-check `show spanning-tree` and confirm traffic recovers

---

## Part 10 — Common Mistakes & Exam Traps

| Wrong idea | Correct idea |
| ---------- | ------------ |
| “STP prevents all loops forever with no config” | Defaults work, but **root placement** and edge features matter in real designs |
| “Blocking port is broken” | It is **intentionally** not forwarding to prevent a loop |
| “PortFast on every port” | PortFast is for **host/edge** ports only |
| “One STP for the whole switch always” | Cisco **PVST+/Rapid PVST+** = **per VLAN** |
| “Lowest MAC always wins root” | Lowest **Bridge ID** wins — priority is compared first |
| “STP is Layer 3” | STP is **Layer 2**; routers/SVIs are a different problem space |

---

## Part 11 — Verification Commands

| Command | Use |
| ------- | --- |
| `show spanning-tree` | Roles, states, root per VLAN |
| `show spanning-tree vlan 10` | Focus one VLAN |
| `show spanning-tree root` | Quick root summary |
| `show spanning-tree interface fa0/1` | One port detail |
| `spanning-tree mode rapid-pvst` | Set rapid per-VLAN mode |
| `spanning-tree vlan 10 root primary` | Make this switch preferred root |

---

## Part 12 — Practice Questions (Self-Check)

1. What disaster does STP prevent on a switched LAN with redundant links?
2. What two parts make up a Bridge ID?
3. What is a root port?
4. Classic STP states that do **not** forward user frames include which ones?
5. What does PortFast do, and where should it be applied?
6. What is the difference between PVST+ and a single shared STP instance for all VLANs?
7. Why use Rapid PVST+ instead of classic STP in modern labs?

### Answers

1. **Layer 2 loops** (broadcast storms, MAC flapping, duplicate frames).
2. **Priority + MAC address**.
3. On a non-root switch, the port with the **best path to the root bridge**.
4. **Blocking**, **Listening**, **Learning** (and Disabled).
5. Speeds edge ports to forwarding; apply on **access ports to end hosts**, usually with **BPDU Guard**.
6. **PVST+** runs STP **per VLAN**, enabling per-VLAN root and better uplink load sharing.
7. **Faster convergence** using RSTP mechanisms instead of long 802.1D timers.

---

## Part 13 — Quick Reference Card

```text
Problem: L2 loops + flooding = broadcast storm
Fix: STP builds a loop-free tree (block some ports)

Root = lowest Bridge ID (priority + MAC)
Root Port = best path to root (non-root switches)
Designated Port = best port on a segment
Other ports = Blocking / Alternate

Classic STP: Blocking → Listening → Learning → Forwarding (slow)
RSTP: Discarding / Learning / Forwarding (fast handshake)

Cisco: Rapid PVST+ = RSTP per VLAN
Edge: PortFast + BPDU Guard (hosts only)
Ops: place root at distribution/core on purpose
```

---

**Mastery check:** Build two switches with two links between them, run `show spanning-tree`, identify the Alternate/Blocking port, unplug the forwarding uplink, and watch the blocked port take over — that single lab ties this file together.
