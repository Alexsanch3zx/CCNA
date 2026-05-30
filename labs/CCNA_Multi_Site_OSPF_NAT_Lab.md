# CCNA Packet Tracer Lab – Multi-Site Routing, OSPF & NAT

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

---

# Network Topology

```text
                    [ Cloud / ISP ]
                    203.0.113.0/30
                           |
                      R-ISP (2911)
                           |
                      10.0.0.0/30
                           |
                      R-HQ (2911) ---- 172.16.0.0/30 ---- R-Branch (2911)
                           |                                      |
                    192.168.1.0/24                          192.168.2.0/24
                           |                                      |
                      SW-HQ (2960)                          SW-Branch (2960)
                      /    |    \                             /    |    \
                    PC1  PC2  PC3                           PC4  PC5  PC6
                  (Mgmt) (Eng) (Guest)                    (Sales)(Sales)(IT)
```

**Story:** HQ and Branch need to reach each other over a WAN. Only **HQ** may use the Internet through the ISP. Branch uses HQ as a path for Internet-bound traffic (optional stretch goal) or only reaches internal networks—your choice in Step 8.

---

# Devices Needed

- 3 Routers (2911): `R-ISP`, `R-HQ`, `R-Branch`
- 2 Switches (2960): `SW-HQ`, `SW-Branch`
- 6 PCs
- 1 **Cloud** or **Server** device (represents the Internet / public host)

---

# Addressing Plan

## WAN Links

| Link | Network | R-HQ | R-Branch | R-ISP |
|------|---------|------|----------|-------|
| HQ ↔ ISP | 10.0.0.0/30 | 10.0.0.1 | — | 10.0.0.2 |
| HQ ↔ Branch | 172.16.0.0/30 | 172.16.0.1 | 172.16.0.2 | — |
| ISP ↔ Internet | 203.0.113.0/30 | — | — | 203.0.113.1 (toward cloud) |

Cloud / public test host: **203.0.113.10/24** (gateway 203.0.113.1 on ISP side).

## LANs (no VLANs in this lab—focus on routing)

| Site | Network | Gateway | Sample PCs |
|------|---------|---------|------------|
| HQ – Management | 192.168.1.0/24 | 192.168.1.1 | PC1 .10 |
| HQ – Engineering | 192.168.10.0/24 | 192.168.10.1 | PC2 .10 |
| HQ – Guest | 192.168.99.0/24 | 192.168.99.1 | PC3 .10 |
| Branch – Sales | 192.168.2.0/24 | 192.168.2.1 | PC4 .10, PC5 .11 |
| Branch – IT | 192.168.20.0/24 | 192.168.20.1 | PC6 .10 |

## Loopbacks (OSPF router ID practice)

| Router | Loopback | Purpose |
|--------|----------|---------|
| R-HQ | 1.1.1.1/32 | Stable OSPF router ID |
| R-Branch | 2.2.2.2/32 | Stable OSPF router ID |
| R-ISP | 3.3.3.3/32 | Optional; static routes only |

Use **GigabitEthernet** for LANs and **Serial** or **Gigabit** for WAN in Packet Tracer—match cable types to interface types.

---

# Step 1: Basic Router Interface Configuration

On each router, configure hostnames, interfaces, and `no shutdown`.

**R-HQ example:**

```cisco
enable
configure terminal
hostname R-HQ

interface g0/0
description LAN-Mgmt
ip address 192.168.1.1 255.255.255.0
no shutdown

interface g0/1
description LAN-Eng
ip address 192.168.10.1 255.255.255.0
no shutdown

interface g0/2
description LAN-Guest
ip address 192.168.99.1 255.255.255.0
no shutdown

interface s0/0/0
description WAN-to-Branch
ip address 172.16.0.1 255.255.255.252
clock rate 128000
no shutdown

interface s0/0/1
description WAN-to-ISP
ip address 10.0.0.1 255.255.255.252
no shutdown
```

Configure **R-Branch** and **R-ISP** to match the addressing table. On the DCE side of each serial link, set `clock rate` (Packet Tracer often needs this on one end).

Verify:

```cisco
show ip interface brief
ping 172.16.0.2
```

---

# Step 2: Loopbacks and Router IDs

```cisco
interface loopback 0
ip address 1.1.1.1 255.255.255.255
```

On **R-Branch**, use `2.2.2.2`. These addresses should appear in OSPF later.

---

# Step 3: Static Routes (Baseline Connectivity)

Before OSPF, prove reachability with static routes.

On **R-Branch**, to reach HQ Management LAN:

```cisco
ip route 192.168.1.0 255.255.255.0 172.16.0.1
```

On **R-HQ**, to reach Branch Sales:

```cisco
ip route 192.168.2.0 255.255.255.0 172.16.0.2
```

Test with `ping` from PC4 to PC1. Then remove these static routes in Step 4 when OSPF is working (or leave one as a floating static backup—optional).

**CCNA concept:** Administrative distance—static (1) vs OSPF (110).

---

# Step 4: Configure OSPFv2 (Single Area 0)

Enable OSPF process **1** on R-HQ and R-Branch. Advertise all internal LANs and the WAN link between them.

**R-HQ:**

```cisco
router ospf 1
router-id 1.1.1.1
network 192.168.1.0 0.0.0.255 area 0
network 192.168.10.0 0.0.0.255 area 0
network 192.168.99.0 0.0.0.255 area 0
network 172.16.0.0 0.0.0.3 area 0
```

**R-Branch:**

```cisco
router ospf 1
router-id 2.2.2.2
network 192.168.2.0 0.0.0.255 area 0
network 192.168.20.0 0.0.0.255 area 0
network 172.16.0.0 0.0.0.3 area 0
```

Verify:

```cisco
show ip ospf neighbor
show ip route ospf
show ip protocols
```

From **PC6** (Branch IT), ping **PC2** (HQ Engineering). Traffic should use OSPF-learned routes.

**Wildcard mask tip:** `0.0.0.3` matches the /30 WAN subnet 172.16.0.0–172.16.0.3.

---

# Step 5: Default Route to ISP (HQ Only)

On **R-ISP**, configure a static route to your internal supernets or use connected routes only toward HQ.

On **R-HQ**, point to the ISP:

```cisco
ip route 0.0.0.0 0.0.0.0 10.0.0.2
```

Advertise the default into OSPF so Branch learns it:

```cisco
router ospf 1
default-information originate
```

Verify on **R-Branch**:

```cisco
show ip route
```

You should see a default route via OSPF (`O*E2` or similar).

---

# Step 6: NAT / PAT (Overload) at HQ

Inside networks: HQ LANs. Outside: interface toward ISP.

```cisco
access-list 1 permit 192.168.1.0 0.0.0.255
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.99.0 0.0.0.255

interface s0/0/1
ip nat outside

interface g0/0
ip nat inside

interface g0/1
ip nat inside

interface g0/2
ip nat inside

ip nat inside source list 1 interface s0/0/1 overload
```

From **PC2**, ping **203.0.113.10** (cloud host). Verify translations:

```cisco
show ip nat translations
show ip nat statistics
```

**CCNA concept:** PAT uses one public IP (the outside interface) with different port numbers for each inside host.

---

# Step 7: Extended ACL – Restrict Guest Internet

Block **Guest** VLAN (192.168.99.0/24) from reaching the Internet, but allow Engineering and Management.

Example (apply on inside interface toward Guest or on outbound NAT):

```cisco
ip access-list extended GUEST-FILTER
 deny   ip 192.168.99.0 0.0.0.255 any
 permit ip any any

interface g0/2
ip access-group GUEST-FILTER in
```

Test: PC3 cannot ping 203.0.113.10; PC2 can.

---

# Step 8: SSH and VTY Security (All Routers)

```cisco
enable secret CcnaLab2026!

username admin privilege 15 secret CcnaLab2026!

ip domain-name lab.local
crypto key generate rsa modulus 1024

line vty 0 4
 transport input ssh
 login local

ip access-list standard MGMT-SSH
 permit 192.168.1.0 0.0.0.255
 deny any

line vty 0 4
 access-class MGMT-SSH in
```

From **PC1** (Management), SSH to `192.168.1.1`. From **PC4** (Branch), SSH should fail if you only permit 192.168.1.0/24.

---

# Step 9: Optional – Branch Internet via HQ

If you want Branch PCs to reach the Internet through HQ (no NAT on Branch):

- Ensure OSPF default from Step 5 is on Branch.
- NAT only on R-HQ (already done).
- Branch uses default route to HQ; HQ PATs all inside traffic.

Troubleshoot with `traceroute` from PC4 to 203.0.113.10.

---

# Verification Commands

```cisco
show ip route
show ip ospf neighbor
show ip ospf interface brief
show ip nat translations
show access-lists
show running-config | section router ospf
traceroute 203.0.113.10
```

---

# Troubleshooting Challenges

## Scenario 1 – OSPF Neighbor Down

Wrong subnet mask on the HQ–Branch WAN or one interface still `shutdown`. Fix until `show ip ospf neighbor` shows **FULL**.

## Scenario 2 – No Default Route on Branch

Missing `default-information originate` on R-HQ or ACL blocking OSPF. Fix and verify `O*E2` on Branch.

## Scenario 3 – NAT Works One Way Only

Inside/outside reversed on interfaces. Use `show ip nat statistics` and fix `ip nat inside` / `ip nat outside`.

## Scenario 4 – Guest Can Still Reach Internet

ACL applied on wrong interface or wrong direction (`in` vs `out`). Use `show access-lists` hit counts.

---

# Skills Learned

- WAN and multi-router design
- Static vs dynamic routing
- OSPFv2 single-area configuration
- Router ID and loopbacks
- Default route injection
- NAT/PAT overload
- Extended ACLs
- SSH and VTY access control
- `traceroute` and routing table analysis

**Maps to CCNA domains:** IP Connectivity, IP Services, Security Fundamentals.
