# Cisco Packet Tracer — Two Small Labs  

Quick solo projects you can finish in **1–3 hours each** after you save your **`.pkt`** and document results. Official tool: **Cisco Packet Tracer** (install per Cisco Networking Academy terms).

---

## Project 1 — Two VLANs, One Router (mini “enterprise” LAN)

### Goal

Separate **staff** and **finance** PCs into **two VLANs**, give each subnet internet-style reachability via a **single router** using **router-on-a-stick**.

### Topology (minimal)

| Device | Role |
|--------|------|
| 1× **2960-class** switch | Access + trunk |
| 1× **2911 / 1941** router (or any with 1 LAN interface you can divide) | Default gateway + inter-VLAN |
| 2× **PC** in VLAN 10 (Staff), 2× **PC** in VLAN 20 (Finance) | End hosts |

Sketch: both VLANs terminate on **one physical link** router ↔ switch using a **dot1Q trunk** (*router-on-a-stick*).

### Address plan (example — change subnets if you like)

| VLAN | Name   | Network        | Gateway (router subif) |
|------|--------|----------------|------------------------|
| 10   | Staff  | `192.168.10.0/24` | `192.168.10.1` |
| 20   | Finance| `192.168.20.0/24` | `192.168.20.1` |

### Tasks

1. On the switch: create VLANs **10** and **20**; assign access ports per PC; configure **Gi0/x** toward the router as a **trunk** (same **native VLAN** on both sides on purpose—or document if you intentionally change it).  

2. On the router: one physical interface divided into **`encapsulation dot1Q 10`** and **`20`** subinterfaces with the gateway IPs above; **no shutdown** parent interface.  

3. Set each PC’s **IP**, **mask `/24`**, and **default gateway** to its VLAN gateway.  

4. **Verify:** all four PCs **ping** every other PC (cross-VLAN).  

5. **Optional stretch:** add a **loopback** on the router (e.g. `10.0.0.1/32`) and ping it from a PC—confirms routing is working beyond directly connected nets.

### What to turn in (for yourself)

- Screenshot or export of **logical topology**.  
- Table: **PC name → IP → VLAN**.  
- Output snippets: `show vlan brief`, `show ip int brief` (router), one successful **ping**.

### Common mistakes

- Trunk not carrying both VLANs or access port in wrong VLAN.  
- Subinterface **dot1Q** number doesn’t match VLAN ID.  
- PC **default gateway** missing or wrong subnet.

---

## Project 2 — Two Sites + Static Routes + Outbound PAT

### Goal

Connect **Site A** and **Site B** with **two routers** back-to-back (or three routers: A — B — “ISP stub”). Use **static routes** so each site reaches the other’s LAN. Add **PAT (overload)** so Site A’s PCs can reach a small **“internet”** stub behind the third router (or the far side of Router B if you only use two routers—pick one story and stick to it).

### Topology A — **Two routers only** (simplest)

```
[PC][PC] -- LAN A -- Router1 ---- Router2 -- LAN B -- [PC][PC]
```

### Topology B — **Three routers** (clearer PAT story)

```
Site A LAN -- R1 --- R2 (middle) --- R3 -- stub "ISP" LAN (loopback or 1 PC)
Site B LAN -- also connected somehow to R2 (add third interface or second link as your lab allows).
```

Adapt to the **interfaces your PT devices** expose; draw your final diagram before configuring.

### Address plan (example)

| Segment        | Network              | Router interface idea |
|----------------|----------------------|------------------------|
| Site A LAN     | `10.1.1.0/24`       | R1 G0/0 |
| Site B LAN     | `10.2.2.0/24`       | R2 G0/1 |
| R1 ↔ R2 link   | `172.16.12.0/30`    | R1 `.1`, R2 `.2` |

If you use a **stub ISP**: e.g. `203.0.113.0/29` on R3-facing link; PAT **outside** faces that direction.

### Tasks

1. Configure host IPs and **default gateways** on PCs (gateway = their site router’s LAN address).  

2. On **R1** and **R2**, assign IPs; bring interfaces **up**.  

3. Add **mutual static routes**:  
   - R1 knows how to reach `10.2.2.0/24` **via next-hop** (R2’s address on the link).  
   - R2 knows `10.1.1.0/24` **via next-hop** (R1 on the link).  

4. **Verify:** ping **cross-site** (PC.A → PC.B).  

5. **Add PAT:** On the router that owns the path to “outside,” configure **overload** NAT from **inside** (your LANs) to **outside** toward the stub; use an **extended ACL** matching **RFC1918** sources (or explicit LAN subnets) going outbound.  

6. **Verify NAT:** Place a Server or PC on stub with **HTTP** or **ICMP** reachable; from Site A ping or browse stub; capture `show ip nat translations` (**clear** translations between tests if you need a clean slate).

### What to turn in

- Rough **diagram** with all subnets labeled.  
- `show ip route` from **both** site routers highlighting statics.  
- One **successful** PAT proof (`show ip nat translations` or statistics).

### Common mistakes

- Static route toward **wrong next-hop** (must be reachable on your local subnet).  

- **`ip nat inside` / `ip nat outside`** on wrong interfaces.  

- ACL that **does not match** the traffic you think—**implicit deny** at end.

---

## Quick etiquette

Experiment only on **your own** Packet Tracer files or academy sandboxes—not on real networks unless you’re authorized.
