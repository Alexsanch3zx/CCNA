# CCNA Packet Tracer Lab – Multi-Site Routing, OSPF & NAT

**Do steps in order.** After each CLI step, run **`write memory`** on that device. Save your `.pkt` when done.

---

## Lab Overview

This lab is different from the small office VLAN lab. You build a **two-site WAN** with a simulated ISP and learn routing and edge connectivity skills that appear often on the CCNA exam:

- Multi-router topologies
- Serial / point-to-point WAN links
- Static and default routes
- **OSPFv2** (single area)
- Router IDs and loopback interfaces
- **NAT / PAT** (overload) for Internet access
- Inside / outside NAT interfaces
- Extended ACLs for management traffic
- **SSH** hardening on routers
- Route verification and path troubleshooting

**Prerequisite:** Comfortable with IP addressing, `show ip interface brief`, and basic router CLI.

**Story:** HQ and Branch need to reach each other over a WAN. Only **HQ** performs NAT to the Internet through the ISP. Branch PCs reach the Internet through HQ (default route via OSPF + PAT at HQ).

---

## Before Every CLI Step

You must be in config mode. If you see `Switch>` or `Router>`:

```cisco
enable
configure terminal
```

Paste the commands for that step. When the block ends with `end`, run:

```cisco
write memory
```

That saves the config so it survives a reload.

---

## Quick Reference

### WAN Links

| Link | Network | R-HQ | R-Branch | R-ISP |
| ---- | ------- | ---- | -------- | ----- |
| HQ ↔ ISP | 10.0.0.0/30 | 10.0.0.1 (s0/0/1) | — | 10.0.0.2 (s0/0/0) |
| HQ ↔ Branch | 172.16.0.0/30 | 172.16.0.1 (s0/0/0) | 172.16.0.2 (s0/0/0) | — |
| ISP ↔ Cloud | 203.0.113.0/24 | — | — | 203.0.113.1 (g0/0) |

Public test host (Cloud or Server): **203.0.113.10/24**, gateway **203.0.113.1**.

### LANs (no VLANs — one subnet per router interface)

| Site | Network | Gateway (router) | PCs |
| ---- | ------- | ---------------- | --- |
| HQ – Management | 192.168.1.0/24 | 192.168.1.1 (R-HQ g0/0) | PC1 .10 |
| HQ – Engineering | 192.168.10.0/24 | 192.168.10.1 (R-HQ g0/1) | PC2 .10 |
| HQ – Guest | 192.168.99.0/24 | 192.168.99.1 (R-HQ g0/2) | PC3 .10 |
| Branch – Sales | 192.168.2.0/24 | 192.168.2.1 (R-Branch g0/0) | PC4 .10, PC5 .11 |
| Branch – IT | 192.168.20.0/24 | 192.168.20.1 (R-Branch g0/1) | PC6 .10 |

### Loopbacks (OSPF router ID)

| Router | Loopback | Purpose |
| ------ | -------- | ------- |
| R-HQ | 1.1.1.1/32 | OSPF router ID |
| R-Branch | 2.2.2.2/32 | OSPF router ID |
| R-ISP | 3.3.3.3/32 | Optional; static routes only |

### PC Addressing (static — Desktop → IP Configuration)

| PC | Site | IP | Mask | Gateway |
| -- | ---- | -- | ---- | ------- |
| PC1 | HQ Mgmt | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| PC2 | HQ Eng | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| PC3 | HQ Guest | 192.168.99.10 | 255.255.255.0 | 192.168.99.1 |
| PC4 | Branch Sales | 192.168.2.10 | 255.255.255.0 | 192.168.2.1 |
| PC5 | Branch Sales | 192.168.2.11 | 255.255.255.0 | 192.168.2.1 |
| PC6 | Branch IT | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 |

---

## Step 0 — Build and Cable

**Devices:** 3 routers (2911), 2 switches (2960), 6 PCs, 1 **Cloud** device (or Server as public host).

Rename devices in Packet Tracer: `R-HQ`, `R-Branch`, `R-ISP`, `SW-HQ`, `SW-Branch`, `PC1`–`PC6`.

### Cabling table

| From | Port | Cable type | To | Port |
| ---- | ---- | ---------- | -- | ---- |
| R-HQ | S0/0/0 | **Serial DCE** | R-Branch | S0/0/0 |
| R-HQ | S0/0/1 | **Serial DCE** | R-ISP | S0/0/0 |
| R-ISP | Gig0/0 | Copper Straight-Through | Cloud | (any Ethernet port) |
| R-HQ | Gig0/0 | Copper Straight-Through | SW-HQ | Fa0/24 |
| SW-HQ | Fa0/1 | Copper Straight-Through | PC1 | Fa0 |
| R-HQ | Gig0/1 | Copper Straight-Through | PC2 | Fa0 |
| R-HQ | Gig0/2 | Copper Straight-Through | PC3 | Fa0 |
| R-Branch | Gig0/0 | Copper Straight-Through | SW-Branch | Fa0/24 |
| SW-Branch | Fa0/1 | Copper Straight-Through | PC4 | Fa0 |
| SW-Branch | Fa0/2 | Copper Straight-Through | PC5 | Fa0 |
| R-Branch | Gig0/1 | Copper Straight-Through | PC6 | Fa0 |

**Serial DCE:** On Packet Tracer, the **DCE** end of the cable connects to **R-HQ** on both WAN links. You will set `clock rate 128000` on R-HQ serial interfaces in Step 1.

**Design note:** HQ has three LAN subnets on **three separate router interfaces** (no VLANs). Engineering and Guest PCs connect **directly** to R-HQ g0/1 and g0/2. Only Management uses SW-HQ.

### ASCII overview

```text
                    [ Cloud / Internet ]
                    203.0.113.0/24
                           |
                      R-ISP (2911)
                     s0/0/0 10.0.0.2
                           |
                      10.0.0.0/30
                           |
                      R-HQ (2911) ---- 172.16.0.0/30 ---- R-Branch (2911)
                     g0/0 Mgmt .1.0/24              g0/0 Sales .2.0/24
                     g0/1 Eng  .10.0/24             g0/1 IT   .20.0/24
                     g0/2 Guest .99.0/24                    |
                           |                            SW-Branch
                      SW-HQ / PC2 / PC3                  PC4 PC5 PC6
                         PC1
```

### Full diagram (interfaces, WAN links, NAT, OSPF, legend)

- Source: [CCNA_Multi_Site_OSPF_NAT_Topology.puml](CCNA_Multi_Site_OSPF_NAT_Topology.puml)
- Image: [CCNA_Multi_Site_OSPF_NAT_Topology.png](CCNA_Multi_Site_OSPF_NAT_Topology.png)

Render PNG: `plantuml labs/CCNA_Multi_Site_OSPF_NAT_Topology.puml` or paste the `.puml` file into [PlantUML online](https://www.plantuml.com/plantuml).

---

## Step 1 — PC and Cloud Addressing (GUI)

On each PC: **Desktop → IP Configuration → Static** — use the Quick Reference table above.

**Cloud (or Server as public host):**

| Setting | Value |
| ------- | ----- |
| IP | 203.0.113.10 |
| Mask | 255.255.255.0 |
| Gateway | 203.0.113.1 |

If using a **Server** device instead of Cloud, cable it to R-ISP Gig0/0 and use the same IP settings.

**Check:** From PC1, `ping 192.168.1.1` — should fail until Step 2 (router not configured yet). After Step 2, all PCs should ping their gateway.

---

## Step 2 — Router Interfaces (R-HQ, R-Branch, R-ISP)

### R-HQ

```cisco
enable
configure terminal
hostname R-HQ

interface g0/0
 description LAN-Mgmt
 ip address 192.168.1.1 255.255.255.0
 no shutdown
exit
interface g0/1
 description LAN-Eng
 ip address 192.168.10.1 255.255.255.0
 no shutdown
exit
interface g0/2
 description LAN-Guest
 ip address 192.168.99.1 255.255.255.0
 no shutdown
exit
interface s0/0/0
 description WAN-to-Branch
 ip address 172.16.0.1 255.255.255.252
 clock rate 128000
 no shutdown
exit
interface s0/0/1
 description WAN-to-ISP
 ip address 10.0.0.1 255.255.255.252
 clock rate 128000
 no shutdown
exit
end
write memory
```

### R-Branch

```cisco
enable
configure terminal
hostname R-Branch

interface g0/0
 description LAN-Sales
 ip address 192.168.2.1 255.255.255.0
 no shutdown
exit
interface g0/1
 description LAN-IT
 ip address 192.168.20.1 255.255.255.0
 no shutdown
exit
interface s0/0/0
 description WAN-to-HQ
 ip address 172.16.0.2 255.255.255.252
 no shutdown
exit
end
write memory
```

### R-ISP

```cisco
enable
configure terminal
hostname R-ISP

interface g0/0
 description To-Cloud
 ip address 203.0.113.1 255.255.255.0
 no shutdown
exit
interface s0/0/0
 description WAN-to-HQ
 ip address 10.0.0.2 255.255.255.252
 no shutdown
exit
end
write memory
```

**Check — on R-HQ:**

```cisco
show ip interface brief
ping 172.16.0.2
ping 10.0.0.2
```

**Expected:** Serial and Gig interfaces **up/up**. Pings to Branch and ISP WAN IPs succeed.

**Check — on each PC:** `ping` your gateway (e.g. PC1 → `192.168.1.1`).

---

## Step 3 — Loopbacks and Router IDs

### R-HQ

```cisco
enable
configure terminal
interface loopback 0
 ip address 1.1.1.1 255.255.255.255
exit
end
write memory
```

### R-Branch

```cisco
enable
configure terminal
interface loopback 0
 ip address 2.2.2.2 255.255.255.255
exit
end
write memory
```

### R-ISP (optional)

```cisco
enable
configure terminal
interface loopback 0
 ip address 3.3.3.3 255.255.255.255
exit
end
write memory
```

**Check:** `show ip interface brief` — Loopback0 shows the correct /32 on each router.

---

## Step 4 — Static Routes (Baseline Connectivity)

Before OSPF, prove reachability with static routes.

### R-Branch

```cisco
enable
configure terminal
ip route 192.168.1.0 255.255.255.0 172.16.0.1
ip route 192.168.10.0 255.255.255.0 172.16.0.1
ip route 192.168.99.0 255.255.255.0 172.16.0.1
end
write memory
```

### R-HQ

```cisco
enable
configure terminal
ip route 192.168.2.0 255.255.255.0 172.16.0.2
ip route 192.168.20.0 255.255.255.0 172.16.0.2
end
write memory
```

**Test:** From **PC4**, `ping 192.168.1.10` (PC1) — **4 replies**.

**CCNA concept:** Administrative distance — static (1) vs OSPF (110).

You will remove these statics in Step 5 after OSPF is working.

---

## Step 5 — Configure OSPFv2 (Single Area 0)

Enable OSPF process **1** on R-HQ and R-Branch. Advertise all internal LANs and the HQ–Branch WAN link.

### R-HQ

```cisco
enable
configure terminal
router ospf 1
 router-id 1.1.1.1
 network 192.168.1.0 0.0.0.255 area 0
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.99.0 0.0.0.255 area 0
 network 172.16.0.0 0.0.0.3 area 0
exit
end
write memory
```

### R-Branch

```cisco
enable
configure terminal
router ospf 1
 router-id 2.2.2.2
 network 192.168.2.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 172.16.0.0 0.0.0.3 area 0
exit
end
write memory
```

### Remove static routes (replaced by OSPF)

**On R-HQ:**

```cisco
enable
configure terminal
no ip route 192.168.2.0 255.255.255.0 172.16.0.2
no ip route 192.168.20.0 255.255.255.0 172.16.0.2
end
write memory
```

**On R-Branch:**

```cisco
enable
configure terminal
no ip route 192.168.1.0 255.255.255.0 172.16.0.1
no ip route 192.168.10.0 255.255.255.0 172.16.0.1
no ip route 192.168.99.0 255.255.255.0 172.16.0.1
end
write memory
```

**Check — on R-HQ or R-Branch:**

```cisco
show ip ospf neighbor
show ip route ospf
```

**Expected neighbor:** State **FULL** on 172.16.0.x. OSPF routes (`O`) for remote LANs.

**Wildcard mask tip:** `0.0.0.3` matches the /30 WAN subnet 172.16.0.0–172.16.0.3.

**Test:** From **PC6** (Branch IT), `ping 192.168.10.10` (PC2) — **4 replies** via OSPF.

---

## Step 6 — Default Route to ISP (HQ + ISP)

### R-ISP — static routes to internal networks via HQ

```cisco
enable
configure terminal
ip route 192.168.0.0 255.255.0.0 10.0.0.1
end
write memory
```

### R-HQ — default route toward ISP

```cisco
enable
configure terminal
ip route 0.0.0.0 0.0.0.0 10.0.0.2
end
write memory
```

### R-HQ — advertise default into OSPF

```cisco
enable
configure terminal
router ospf 1
 default-information originate
exit
end
write memory
```

**Check — on R-Branch:**

```cisco
show ip route
```

**Expected:** Default route via OSPF (`O*E2` or similar), pointing toward 172.16.0.1.

**Check — on R-HQ:**

```cisco
ping 203.0.113.10
```

Should succeed before NAT (HQ routes to ISP, ISP routes back).

---

## Step 7 — NAT / PAT (Overload) at HQ

Inside networks: HQ LANs. Outside: interface toward ISP.

```cisco
enable
configure terminal
access-list 1 permit 192.168.1.0 0.0.0.255
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.99.0 0.0.0.255
interface s0/0/1
 ip nat outside
exit
interface g0/0
 ip nat inside
exit
interface g0/1
 ip nat inside
exit
interface g0/2
 ip nat inside
exit
ip nat inside source list 1 interface s0/0/1 overload
end
write memory
```

**Test:** From **PC2**, `ping 203.0.113.10`.

**Check — on R-HQ:**

```cisco
show ip nat translations
show ip nat statistics
```

**Expected:** Active translations after ping. Inside local → inside global (10.0.0.1 with port numbers).

**CCNA concept:** PAT uses one public IP (the outside interface) with different port numbers for each inside host.

---

## Step 8 — Extended ACL: Block Guest Internet

Block **Guest** (192.168.99.0/24) from reaching the Internet; Management and Engineering still work.

```cisco
enable
configure terminal
ip access-list extended GUEST-FILTER
 deny ip 192.168.99.0 0.0.0.255 any
 permit ip any any
exit
interface g0/2
 ip access-group GUEST-FILTER in
exit
end
write memory
```

**Test:**

| From | Command | Expected |
| ---- | ------- | -------- |
| PC3 (Guest) | `ping 203.0.113.10` | **Fail** — timed out |
| PC2 (Eng) | `ping 203.0.113.10` | **4 replies** |
| PC3 (Guest) | `ping 192.168.10.10` (PC2) | **4 replies** (internal OK) |

**Check:** `show access-lists` — deny line hit count increases when PC3 tries Internet.

---

## Step 9 — SSH and VTY Security (All Three Routers)

Run the **same block** on **R-HQ**, **R-Branch**, and **R-ISP** (adjust nothing — ACL permits HQ Mgmt subnet only).

```cisco
enable
configure terminal
enable secret CcnaLab2026!
username admin privilege 15 secret CcnaLab2026!
ip domain-name lab.local
crypto key generate rsa modulus 1024
line vty 0 4
 transport input ssh
 login local
exit
ip access-list standard MGMT-SSH
 permit 192.168.1.0 0.0.0.255
 deny any
exit
line vty 0 4
 access-class MGMT-SSH in
exit
end
write memory
```

When prompted to replace existing keys, answer **yes**.

**Test:**

| From | Target | Expected |
| ---- | ------ | -------- |
| PC1 | `ssh -l admin 192.168.1.1` | **Login succeeds** |
| PC4 | `ssh -l admin 192.168.2.1` | **Fails** (not in MGMT-SSH ACL) |

---

## Step 10 — Branch Internet via HQ

Branch PCs use the OSPF default route from Step 6 and HQ PAT from Step 7. No NAT on R-Branch.

**Test — from PC4:**

```text
ping 203.0.113.10
traceroute 203.0.113.10
```

**Expected traceroute path:** PC4 → 192.168.2.1 (R-Branch) → 172.16.0.1 (R-HQ) → 10.0.0.2 (R-ISP) → 203.0.113.10.

**Check — on R-HQ after Branch ping:**

```cisco
show ip nat translations
```

Translations may show Branch source IPs if you extend ACL 1 (stretch goal). By default, only HQ subnets are in ACL 1 — to allow Branch Internet through HQ NAT, add:

```cisco
enable
configure terminal
access-list 1 permit 192.168.2.0 0.0.0.255
access-list 1 permit 192.168.20.0 0.0.0.255
end
write memory
```

Then retest from PC4.

---

## Step 11 — Save Lab File

**Packet Tracer:** **File → Save As** your `.pkt` file.

(Optional final save on all routers if you skipped `write memory` earlier: `enable` → `write memory`.)

---

## Verification Commands

```cisco
show ip route
show ip ospf neighbor
show ip ospf interface brief
show ip nat translations
show ip nat statistics
show access-lists
show running-config | section router ospf
traceroute 203.0.113.10
```

---

## Done Checklist

| Test | Command / action | Should work? |
| ---- | ---------------- | ------------ |
| PC gateway | PC1: `ping 192.168.1.1` | Yes — 4 replies |
| OSPF neighbor | R-HQ: `show ip ospf neighbor` | FULL on 172.16.0.x |
| Cross-site | PC6: `ping 192.168.10.10` | Yes — OSPF route |
| HQ Internet | PC2: `ping 203.0.113.10` | Yes — NAT/PAT |
| Guest blocked | PC3: `ping 203.0.113.10` | No — ACL deny |
| Guest internal | PC3: `ping 192.168.1.10` | Yes — not blocked |
| Default on Branch | R-Branch: `show ip route` | O*E2 default |
| Branch Internet | PC4: `ping 203.0.113.10` | Yes (after ACL 1 includes Branch) |
| SSH from Mgmt | PC1: `ssh -l admin 192.168.1.1` | Yes |
| SSH from Branch | PC4: `ssh -l admin 192.168.2.1` | No |

---

## Troubleshooting Challenges

### Scenario 1 — OSPF Neighbor Down

Wrong subnet mask on the HQ–Branch WAN or one interface still `shutdown`. Fix until `show ip ospf neighbor` shows **FULL**.

### Scenario 2 — No Default Route on Branch

Missing `default-information originate` on R-HQ or static default missing on R-HQ. Fix and verify `O*E2` on Branch.

### Scenario 3 — NAT Works One Way Only

Inside/outside reversed on interfaces. Use `show ip nat statistics` and fix `ip nat inside` / `ip nat outside`.

### Scenario 4 — Guest Can Still Reach Internet

ACL applied on wrong interface or wrong direction (`in` vs `out`). Use `show access-lists` hit counts.

### Scenario 5 — Serial Link Down

DCE cable on wrong router or missing `clock rate` on R-HQ. Use `show controllers serial` and `show ip interface brief`.

---

## Quick Fixes

| Problem | Fix |
| ------- | --- |
| `Invalid input` on `interface` | Run `enable` then `configure terminal` first |
| Serial interface down/down | DCE on R-HQ; `clock rate 128000` on R-HQ s0/0/0 and s0/0/1 |
| PCs can't ping gateway | Check Step 2 IPs; `no shutdown` on router interface |
| No OSPF neighbor | Matching /30 on both ends; both s0/0/0 up/up |
| HQ can't ping Cloud | R-ISP g0/0 up; static route on ISP; Cloud IP .10 GW .1 |
| NAT no translations | `ip nat inside` on LANs, `outside` on s0/0/1; ACL 1 permits source |
| Branch no Internet | Default on Branch; extend ACL 1 for 192.168.2.0 and 192.168.20.0 |
| Config lost after reload | Run `write memory` after each step |

---

## Skills Learned

- WAN and multi-router design
- Serial cabling (DCE/DTE, clock rate)
- Static vs dynamic routing
- OSPFv2 single-area configuration
- Router ID and loopbacks
- Default route injection
- NAT/PAT overload
- Extended ACLs
- SSH and VTY access control
- `traceroute` and routing table analysis

**Maps to CCNA domains:** IP Connectivity, IP Services, Security Fundamentals.
