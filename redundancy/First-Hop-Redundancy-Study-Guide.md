# First-Hop Redundancy (HSRP / VRRP / GLBP)

**Audience:** CCNA students who need to understand **default-gateway redundancy** — why a single gateway IP is a single point of failure, and how FHRPs provide a **virtual gateway**.

**Folder:** `redundancy/` — high-availability gateway topics (pairs with STP/EtherChannel physical redundancy).

**Related files:** [../terminology.md](../terminology.md) · [../MAC-addresses.md](../MAC-addresses.md) · [../inter_vlan_routing.md](../inter_vlan_routing.md) · [../cheat-sheets/command-cheat-sheet.md](../cheat-sheets/command-cheat-sheet.md) · [../labs/CCNA_Layer3_Switching_Redundancy_Lab.md](../labs/CCNA_Layer3_Switching_Redundancy_Lab.md)

---

## Part 1 — What Problem Does FHRP Solve?

Hosts usually have **one default gateway** (e.g. `192.168.10.1`). If that router or L3 switch dies:

- LAN hosts still have IPs
- Same-subnet traffic may work
- **Off-subnet / Internet traffic fails**

**First Hop Redundancy Protocols (FHRPs)** let two (or more) routers share a **virtual IP** (and virtual MAC) that hosts use as the gateway. If the active device fails, a standby takes over — hosts keep the **same gateway IP**.

```text
PC default gateway = 192.168.10.1  (virtual IP)

   R1 (Active)  ----+----  R2 (Standby)
   .2 physical      |      .3 physical
                    |
              VIP .1  ← PCs point here
```

**CCNA takeaway:** FHRP protects the **first hop** (the gateway), not the whole WAN path by itself.

---

## Part 2 — Shared Ideas Across HSRP / VRRP / GLBP

| Concept | Meaning |
| ------- | ------- |
| **Virtual IP (VIP)** | Gateway address configured on hosts |
| **Virtual MAC** | MAC used for the VIP (protocol-specific pattern) |
| **Active / Master** | Device currently forwarding for the VIP |
| **Standby / Backup** | Ready to take over |
| **Hello / Advertisement** | Keepalive messages between peers |
| **Priority** | Higher priority preferred for active role |
| **Preemption** | Higher-priority device can reclaim active when it returns |
| **Tracking** | Lower priority if a tracked interface/route fails |

Hosts learn the virtual MAC via **ARP** for the VIP (often after failover, gratuitous ARP updates the path).

---

## Part 3 — HSRP (Hot Standby Router Protocol)

**Cisco proprietary.** Classic CCNA FHRP.

### 3.1 Roles

| Role | Job |
| ---- | --- |
| **Active** | Forwards packets for the VIP; answers ARP for VIP |
| **Standby** | Monitors active; takes over on failure |
| **Listen** (others) | Know about the group but are neither active nor standby |

### 3.2 Versions (awareness)

| Version | Notes |
| ------- | ----- |
| **HSRPv1** | Common in older materials; multicast `224.0.0.2`; group 0–255 |
| **HSRPv2** | Wider group range; different virtual MAC format; multicast `224.0.0.102` |

### 3.3 Virtual MAC pattern (HSRPv1 style — exam favorite)

```text
0000.0c07.acXX
```

`XX` = HSRP group number in hex (e.g. group 10 → `0a`).

### 3.4 Basic config (SVI or router interface)

```cisco
interface vlan 10
 ip address 192.168.10.2 255.255.255.0
 standby 10 ip 192.168.10.1
 standby 10 priority 110
 standby 10 preempt
 standby 10 track GigabitEthernet0/1 20
```

On the peer:

```cisco
interface vlan 10
 ip address 192.168.10.3 255.255.255.0
 standby 10 ip 192.168.10.1
 standby 10 priority 90
 standby 10 preempt
```

| Command piece | Meaning |
| ------------- | ------- |
| `standby 10 ip ...` | Group **10**, VIP |
| `priority` | Default **100**; higher wins (if preempt allows) |
| `preempt` | Take over if you have higher priority |
| `track` | Decrement priority when tracked object fails |

### 3.5 Verify

```cisco
show standby
show standby brief
```

Look for: **Active**, **Standby**, VIP, priority, state (Active/Standby/Speak/Listen/Init).

---

## Part 4 — VRRP (Virtual Router Redundancy Protocol)

**Standards-based** (IETF). Same job as HSRP with slightly different names.

| HSRP term | VRRP term |
| --------- | --------- |
| Active | **Master** |
| Standby | **Backup** |
| `standby` commands | `vrrp` commands |

### Conceptual config

```cisco
interface vlan 10
 ip address 192.168.10.2 255.255.255.0
 vrrp 10 ip 192.168.10.1
 vrrp 10 priority 110
 vrrp 10 preempt
```

**Virtual MAC pattern (common):**

```text
0000.5e00.01XX
```

`XX` = VRID (group) in hex.

**CCNA comparison point:** VRRP is **multi-vendor friendly**; HSRP is **Cisco**. Behavior on the exam is “virtual gateway with master/backup.”

Verify: `show vrrp`, `show vrrp brief`

---

## Part 5 — GLBP (Gateway Load Balancing Protocol)

**Cisco proprietary.** Still provides a virtual IP, but can **load-balance** across multiple routers.

### 5.1 Roles

| Role | Job |
| ---- | --- |
| **AVG** (Active Virtual Gateway) | Answers ARP for the VIP; assigns virtual MACs to forwarders |
| **AVF** (Active Virtual Forwarder) | Actually forwards traffic for a virtual MAC assigned by the AVG |

Hosts still use **one VIP**, but ARP replies may give **different virtual MACs** so traffic is shared.

### 5.2 Why it exists

HSRP/VRRP are typically **active/standby** for a given VIP (standby idle for that VIP unless you use multiple groups). GLBP’s selling point is **load sharing** with one VIP.

### 5.3 Awareness config shape

```cisco
interface vlan 10
 glbp 10 ip 192.168.10.1
 glbp 10 priority 110
 glbp 10 preempt
 glbp 10 load-balancing round-robin
```

Verify: `show glbp`, `show glbp brief`

**Exam depth:** Know **what problem GLBP solves** and AVG/AVF names; HSRP is usually the deep lab protocol.

---

## Part 6 — HSRP vs VRRP vs GLBP (Compare)

| Feature | HSRP | VRRP | GLBP |
| ------- | ---- | ---- | ---- |
| Standard | Cisco | Industry standard | Cisco |
| Active names | Active / Standby | Master / Backup | AVG + AVFs |
| Load balancing | Multiple groups (common design) | Multiple groups | Built-in with one VIP |
| VIP on a real interface IP | Not the physical IP (VIP separate) | Master may own VIP as real IP in some designs | VIP separate |

**Design tip:** With two distribution switches, many designs use **HSRP** (or VRRP) and make **SW1 active for odd VLANs**, **SW2 active for even VLANs** — similar idea to per-VLAN STP root placement for uplink use.

---

## Part 7 — Failover Behavior (What Hosts Experience)

1. Active stops sending hellos (crash, link down, priority drop via tracking).
2. Standby waits hold time, then becomes active.
3. New active may send **gratuitous ARP** for the VIP → switches update CAM to the new physical port/path.
4. Hosts keep gateway `192.168.10.1` — no reconfiguration.

**Tracking example:** If the active router’s WAN interface dies but LAN is up, tracking can lower priority so the standby (with a healthy WAN) takes the VIP.

---

## Part 8 — Lab Tie-In

In [CCNA_Layer3_Switching_Redundancy_Lab.md](../labs/CCNA_Layer3_Switching_Redundancy_Lab.md):

- Two multilayer switches (`DLSW1` / `DLSW2`) already give redundant L2/L3 paths
- Stretch goal: **HSRP on SVIs** so `.1` floats between switches

Minimum proof:

1. Hosts use VIP `.1`
2. `show standby` shows Active on DLSW1, Standby on DLSW2
3. Shut the active SVI or reload Active → Standby becomes Active → ping still works

---

## Part 9 — Common Mistakes & Exam Traps

| Wrong idea | Correct idea |
| ---------- | ------------ |
| “FHRP replaces STP” | STP is **L2 loops**; FHRP is **gateway IP** redundancy |
| “Hosts need two default gateways” | Hosts need **one VIP**; routers share it |
| “Standby forwards normally in HSRP” | Only **Active** forwards for that VIP (unless multiple groups) |
| “Priority alone always forces takeover” | Need **preemption** for a returning higher-priority router to reclaim |
| “Virtual MAC is the router BIA” | VIP uses a **protocol virtual MAC** |

---

## Part 10 — Practice Questions (Self-Check)

1. What address do PCs configure as their default gateway in an HSRP design?
2. What are the two main HSRP roles?
3. What does preemption do?
4. Which FHRP is the industry standard?
5. Which FHRP is known for load balancing with a single VIP?
6. Why might you track an uplink interface?

### Answers

1. The **virtual IP (VIP)**.
2. **Active** and **Standby**.
3. Lets a **higher-priority** router take (or reclaim) the active role.
4. **VRRP**.
5. **GLBP**.
6. So priority drops if the WAN/uplink fails and the peer with a good path can become active.

---

## Part 11 — Quick Reference Card

```text
FHRP = virtual default gateway for hosts

HSRP (Cisco): Active / Standby
  standby <group> ip <VIP>
  priority / preempt / track
  vMAC ~ 0000.0c07.acXX

VRRP (standard): Master / Backup
  vrrp <group> ip <VIP>
  vMAC ~ 0000.5e00.01XX

GLBP (Cisco): AVG + AVFs, load share one VIP

Hosts ARP the VIP → learn virtual MAC
Failover ≈ hello loss → new active + GARP

Verify: show standby | show vrrp | show glbp
```

---

**Mastery check:** Draw two L3 switches, one VIP per VLAN, label Active/Standby, and explain what the PC’s ARP cache holds for the gateway before and after failover.
