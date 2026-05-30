# CCNA Packet Tracer Lab – Layer 3 Switching, EtherChannel & STP

## Lab Overview

The small office lab uses a **router-on-a-stick** for inter-VLAN routing. This lab teaches a different and very exam-relevant approach:

- **Multilayer switches (MLS)** and **SVIs** for inter-VLAN routing
- **DHCP relay** (`ip helper-address`)
- **EtherChannel** (LACP, 802.3ad)
- **STP** root bridge placement and verification
- **PortFast** and **BPDU Guard** on access ports
- **IPv6** addressing on a VLAN (introduction)
- Redundant paths without loops

**Prerequisite:** VLANs, trunking, and basic switch CLI from the small office lab.

---

# Network Topology

```text
              [ Router R1 ]  DHCP / DNS / Default Gateway to "WAN"
                     |
                 g0/0 192.168.255.1/24  (optional uplink)
                     |
    +----------------+----------------+
    |                                 |
 DLSW1 (3560) ===================== DLSW2 (3560)   <- EtherChannel (LACP)
    | \                               / |
    |  \                             /  |
    |   ASW1 (2960)           ASW2 (2960)
    |      |    \               /    |
   PC1    PC2   PC3          PC4   PC5
  VLAN10 VLAN20 VLAN30      VLAN10 VLAN20
```

**Design:** Two distribution switches provide **Layer 3 gateways** for VLANs. Access switches connect end devices. A router (or server) supplies DHCP for multiple subnets via **helper addresses**.

---

# Devices Needed

- 1 Router (2911) – `R1` (DHCP server and optional WAN)
- 2 Multilayer switches (3560 or 3650) – `DLSW1`, `DLSW2`
- 2 Access switches (2960) – `ASW1`, `ASW2`
- 5 PCs
- 1 Server (optional, for DHCP/DNS on router instead)

---

# VLAN Design

| VLAN | Name | Subnet (IPv4) | Gateway (SVI) |
|------|------|---------------|---------------|
| 10 | DATA | 10.10.10.0/24 | 10.10.10.1 on DLSW1, .2 on DLSW2 (HSRP optional) |
| 20 | VOICE | 10.10.20.0/24 | 10.10.20.1 / .2 |
| 30 | MGMT | 10.10.30.0/24 | 10.10.30.1 / .2 |
| 99 | NATIVE | (trunk only) | — |

For simplicity, use **DLSW1** as the active gateway (.1) on each SVI. DLSW2 can have standby IPs if you add HSRP (stretch goal).

---

# IP Addressing (Examples)

| Device | VLAN | IPv4 Address | IPv6 (optional) |
|--------|------|--------------|-----------------|
| PC1 | 10 | 10.10.10.10/24 | 2001:db8:10::10/64 |
| PC2 | 20 | 10.10.20.10/24 | 2001:db8:20::10/64 |
| PC3 | 30 | 10.10.30.10/24 | — |
| PC4 | 10 | 10.10.10.11/24 | — |
| PC5 | 20 | 10.10.20.11/24 | — |
| R1 g0/0 | — | 10.10.255.1/24 | — |

**DHCP pools on R1** (or Server):

- `DATA`: 10.10.10.0/24, gateway 10.10.10.1
- `VOICE`: 10.10.20.0/24, gateway 10.10.20.1
- `MGMT`: 10.10.30.0/24, gateway 10.10.30.1

---

# Step 1: Create VLANs on All Switches

On **DLSW1**, **DLSW2**, **ASW1**, **ASW2**:

```cisco
enable
configure terminal
hostname DLSW1

vlan 10
 name DATA
vlan 20
 name VOICE
vlan 30
 name MGMT
vlan 99
 name NATIVE
```

Repeat on other switches (adjust hostname).

---

# Step 2: Access Ports on ASW1 and ASW2

**ASW1:**

```cisco
interface range fa0/1-2
 switchport mode access
 switchport access vlan 10

interface fa0/3
 switchport mode access
 switchport access vlan 20

interface fa0/4
 switchport mode access
 switchport access vlan 30
```

**ASW2:** Assign PC4 to VLAN 10, PC5 to VLAN 20.

---

# Step 3: Trunks to Distribution Layer

Trunk links: ASW1 ↔ DLSW1, ASW1 ↔ DLSW2, ASW2 ↔ DLSW1, ASW2 ↔ DLSW2 (redundant paths).

On each trunk interface:

```cisco
interface fa0/24
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
```

Use the port numbers that match your cabling. Verify:

```cisco
show interfaces trunk
show vlan brief
```

---

# Step 4: SVIs and Inter-VLAN Routing on DLSW1

Enable IP routing on the multilayer switch:

```cisco
ip routing

interface vlan 10
 ip address 10.10.10.1 255.255.255.0
 no shutdown

interface vlan 20
 ip address 10.10.20.1 255.255.255.0
 no shutdown

interface vlan 30
 ip address 10.10.30.1 255.255.255.0
 no shutdown
```

Configure the same VLANs and **different** gateway IPs on **DLSW2** only if you are using HSRP; otherwise leave DLSW2 SVIs shut down until Step 6, or use the same IPs only on one switch to avoid duplicate IP conflicts.

**Recommended for first pass:** Only **DLSW1** has active SVI gateways; DLSW2 is L2 until EtherChannel and STP are stable.

Test: PC1 pings PC2 (different VLANs) via DLSW1.

---

# Step 5: DHCP Relay (ip helper-address)

Clients live on remote VLANs; the DHCP server is on **R1** at 10.10.255.10 (example).

On **DLSW1**:

```cisco
interface vlan 10
 ip helper-address 10.10.255.10

interface vlan 20
 ip helper-address 10.10.255.10

interface vlan 30
 ip helper-address 10.10.255.10
```

On **R1**, configure DHCP pools and exclude gateway addresses.

Set PCs to DHCP. Verify:

```cisco
show ip dhcp binding
```

On PC: `ipconfig` / Desktop → IP Configuration.

**CCNA concept:** Without `ip helper-address`, DHCP broadcasts do not cross VLAN boundaries.

---

# Step 6: EtherChannel Between DLSW1 and DLSW2

Connect **two** links between DLSW1 and DLSW2 (e.g., `gi1/0/23` and `gi1/0/24`).

**DLSW1:**

```cisco
interface range gi1/0/23-24
 channel-group 1 mode active
 switchport trunk encapsulation dot1q
 switchport mode trunk

interface port-channel 1
 switchport trunk encapsulation dot1q
 switchport mode trunk
```

**DLSW2:**

```cisco
interface range gi1/0/23-24
 channel-group 1 mode passive
 switchport trunk encapsulation dot1q
 switchport mode trunk

interface port-channel 1
 switchport trunk encapsulation dot1q
 switchport mode trunk
```

Verify:

```cisco
show etherchannel summary
show interfaces port-channel 1
```

**CCNA concept:** `mode active` = LACP (802.3ad); both sides must agree (active/passive or active/active per platform).

---

# Step 7: STP – Root Bridge and Verification

Make **DLSW1** the root for all VLANs:

```cisco
spanning-tree vlan 10,20,30 root primary
```

On **DLSW2**:

```cisco
spanning-tree vlan 10,20,30 root secondary
```

Verify:

```cisco
show spanning-tree root
show spanning-tree vlan 10
```

Identify:

- Root bridge ID
- Root port on each non-root switch
- Designated vs blocked ports on redundant links

**CCNA concept:** Only one root bridge per VLAN; blocked ports prevent loops.

---

# Step 8: PortFast and BPDU Guard (Access Ports)

On **ASW1** and **ASW2**, all host-facing access ports:

```cisco
interface range fa0/1-4
 spanning-tree portfast
 spanning-tree bpduguard enable
```

Or globally (where supported):

```cisco
spanning-tree portfast default
spanning-tree portfast bpduguard default
```

Connect a **switch** to an access port (lab mistake simulation)—port should err-disable if BPDU Guard is working.

---

# Step 9: IPv6 on VLAN 10 (Introduction)

On **DLSW1**:

```cisco
interface vlan 10
 ipv6 address 2001:db8:10::1/64
 ipv6 enable
```

On **PC1**, set IPv6 to `2001:db8:10::10/64` with gateway `2001:db8:10::1`.

Ping another IPv6 host or `2001:db8:10::1`.

Verify:

```cisco
show ipv6 interface brief
show ipv6 neighbors
```

**CCNA concept:** Link-local addresses (`fe80::`) are automatic; global unicast routes may need `ipv6 unicast-routing` on L3 devices.

---

# Step 10: Default Route to Router (Optional Uplink)

Connect **R1** to DLSW1. On **DLSW1**:

```cisco
interface vlan 255
 ip address 10.10.255.2 255.255.255.0

ip route 0.0.0.0 0.0.0.0 10.10.255.1
```

On **R1**, add routes back to internal subnets or use a single summary route:

```cisco
ip route 10.10.0.0 255.255.0.0 10.10.255.2
```

---

# Verification Commands

```cisco
show ip route
show ip interface brief
show vlan brief
show interfaces trunk
show etherchannel summary
show spanning-tree summary
show spanning-tree vlan 10 detail
show ip helper-address
show ipv6 interface brief
```

---

# Troubleshooting Challenges

## Scenario 1 – PCs in Same VLAN Cannot Ping

Access port in wrong VLAN or trunk not carrying VLAN 10. Fix with `show vlan brief` and `show interfaces trunk`.

## Scenario 2 – DHCP Fails Across VLANs

Missing `ip helper-address` or wrong server IP. ACL on R1 blocking UDP 67/68 (advanced).

## Scenario 3 – EtherChannel Down

Mode mismatch (on/auto/active/passive) or one member port not in trunk mode. `show etherchannel summary` shows (P) vs (D).

## Scenario 4 – Temporary Loop / Broadcast Storm

STP not converged, or PortFast on a trunk by mistake. Verify root bridge and blocked port.

## Scenario 5 – IPv6 Works Link-Local Only

`ipv6 unicast-routing` not enabled on DLSW1. Enable and retest.

---

# Stretch Goals

1. **HSRP** on DLSW1/DLSW2 so `.1` gateways float between switches.
2. **VTP transparent** mode on access switches while distribution switches use **VTP server** (document revision and VLAN pruning risks).
3. **Port security** on ASW1 matching the small office lab.
4. **Standard ACL** on R1 allowing only MGMT (10.10.30.0/24) to SSH to switches.

---

# Skills Learned

- Layer 3 switching with SVIs
- Inter-VLAN routing without router-on-a-stick
- DHCP relay (`ip helper-address`)
- EtherChannel (LACP)
- STP root bridge election and topology analysis
- PortFast and BPDU Guard
- IPv6 interface configuration
- Redundant switched network design

**Maps to CCNA domains:** Network Access, IP Connectivity, IP Services (IPv6 basics).

---

# How This Lab Complements the Small Office Lab

| Topic | Small Office Lab | This Lab |
|-------|------------------|----------|
| Inter-VLAN routing | Router-on-a-stick | MLS + SVI |
| Routing protocol | None | Optional static to R1 |
| Redundancy | Single trunk | EtherChannel + STP |
| DHCP | On router | Central server + relay |
| IPv6 | No | Yes (intro) |

Complete all three labs for broader hands-on coverage before the exam.
