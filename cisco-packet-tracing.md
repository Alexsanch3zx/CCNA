# Cisco Packet Tracer — Extensive Lab Guide

**Audience:** Anyone learning routing, switching, and network design using Cisco’s simulation environment—with enough depth to lab effectively and two **ready-to-build project briefs**.

**Note:** Cisco **Packet Tracer** is officially distributed through the **Cisco Networking Academy**. Use only legitimate installs tied to Cisco’s licensing terms.

---

## Part 1 — What Packet Tracer Is (and Is Not)

### 1.1 Purpose

Packet Tracer is a **visual network emulator** used to:

- **Place** routers, switches, servers, PCs, IoT-ish devices, and cables on a workspace.  
- **Configure** many devices via **CLI** similar to IOS (command set is **subset** and sometimes **behaviorally simplified** vs physical gear or **Cisco Modeling Labs**).  
- **Observe** simplified **packets** traversing links (inspect modes).  
- **Practice** layered troubleshooting without hardware cost.

### 1.2 Strengths for Learning

- Very fast topology iteration (**drag, connect, undo**).  
- Built-in **Activity Wizard** scenarios in academy courses (if provided).  
- Good for **VLAN/trunk**, **basic OSPF/EIGRP (where supported)**, **NAT**, **DHCP relay**, introductory **ACLs**, **WLAN** basics (version-dependent).

### 1.3 Limits You Should Memorize Early


| Limitation                 | Why it matters                                                                  |
| -------------------------- | ------------------------------------------------------------------------------- |
| IOS command/feature subset | Scripts from real jobs may fail in PT—you learn **patterns**, not vendor parity |
| Behavioral simplifications | QoS granularity, advanced STP edge cases, some crypto—often shallow             |
| Not a pen-test oracle      | Attack realism is constrained; Pair with **ethical lab policy** separately      |
| Version drift              | Older PT versions lack newer device models or WLC-style behaviors               |


Treat Packet Tracer as a **guided flight simulator**. When you plateau, augment with **physical lab**, **CML**, **EVE‑NG**, or **cloud sandboxes**.

---

## Part 2 — Getting Oriented

### 2.1 Common Interface Regions

Depending on Packet Tracer version, you typically have:

1. **Logical / Physical workspace** tabs — Logical is schematic; Physical can place racks/cities for multi-site stories.
2. **Device-type palette** bottom-left — routers, switches, hubs, bridges, PCs, servers, cables.
3. **Connections** copper/fiber/console — auto cable choice sometimes works; explicit choice teaches media rules.
4. **Simulation vs Real-Time** toggle — Simulation steps packet events; Real-Time behaves continuously.

### 2.2 Device Boot & Access

New routers/switches boot like small IOS devices—wait until **CLI prompt** responds to input.

**Access modes:**


| Access                                                            | Typical use                             |
| ----------------------------------------------------------------- | --------------------------------------- |
| **Console** terminal                                              | Configure from “out-of-band” user story |
| **Telnet/Ssh** enabled by **VTY** lines + passwords or local user | Remote admin realism                    |
| **GUI** desktop on PCs/servers                                    | Browser to server services              |


### 2.3 CLI Mode Ladder (Cisco IOS Pattern)

Packet Tracer often mirrors classical IOS:

- **User EXEC** → `hostname>`  
- **Privileged EXEC** → `hostname#` (`enable`)  
- **Global config** → `hostname(config)#` (`configure terminal`)  
- **Submodes** — interface, line, router subconfiguration

**Exit discipline:** `exit` steps up one level; `end` or **Ctrl+Z** returns to privileged EXEC.

---

## Part 3 — Cabling & Layer-1 Habits

### 3.1 Straight-Through vs Crossover (Conceptual)

Modern platforms often **auto‑MDIX**, but PT still teaches **device roles**:

- **Switch ↔ PC** — straight-through in classic pedagogy.  
- **Switch ↔ Router (Ethernet)** — straight-through for access-style links.  
- **Router ↔ Router** or **Switch ↔ Switch** historically crossover—PT may automate; **explicit choice** reinforces exam thinking.

### 3.2 Serial WAN Links (Legacy Exam Heritage)

Older curricula use **serial** **DTE/DCE** clocking—know **clock rate** sits on **DCE** side where serial is modeled.

---

## Part 4 — Switching Foundations in Packet Tracer

### 4.1 Baseline Verification

Commands you should reflexively reach for:

```
show vlan brief
show interfaces status
show spanning-tree
show mac address-table
show interfaces trunk
```

### 4.2 VLAN Lab Pattern

**Setup recipe:**

1. Create VLANs (`vlan 10`, `name SALES`).
2. Assign access ports (`switchport mode access`, `switchport access vlan 10`).
3. Trunk uplinks (`switchport mode trunk`, align **native VLAN** deliberately).

**Failure archaeology:** A VLAN missing on one switch—but **allowed on the trunk**—often pairs with VLAN **not defined locally**: classic intermittent reachability drills.

### 4.3 Inter-VLAN Routing Recipes

Pick one story per topology:

**A.** **Router-on-a-stick:** subinterfaces with **802.1Q** encapsulation dot1Q VLAN tags.  

**B.** **Layer 3 switch SVI:** `ip routing` plus SVI IPs on VLAN interfaces.

Verification: **hosts ping across subnets** via correct gateway IPs.

---

## Part 5 — IPv4 Addressing Discipline for PT Labs

Always **draw a table before clicking**:


| Node | Interface | VLAN (if relevant) | IPv4/Mask | Default gateway |
| ---- | --------- | ------------------ | --------- | --------------- |


**Overlapping subnets** miswired between two routers is the silent killer of novice labs—PT will “kind of work locally” yet fail at edges.

Practice **VLSM** on paper first for multi-site labs.

---

## Part 6 — Routing in Packet Tracer

### 6.1 Static Routes

Universal diagnostic:

```
show ip route
show ip interface brief
```

Floating static tricks (Administrative Distance) reinforce **policy routing thinking**.

### 6.2 OSPF Pattern (single-area familiarity)

Mindset checkpoints:

1. Interfaces participate via `**network`** statements—or interface-level activation in newer syntax paths.
2. **Dead/Hello mismatches**, **stub flag conflicts**, **area mismatches**, and **passive-interface** omissions are textbook breakages.

Verification:

```
show ip ospf neighbor
show ip ospf database
show ip protocols
```

### 6.3 EIGRP (Where Available)

If your PT release models it cleanly, compare **composite metric intuition** vs OSPF **cost**.

---

## Part 7 — Services That Make Topologies Feel “Real”

### 7.1 DHCP Server on Generic Server Appliance

Practice **scopes**, **gateway option**, **DNS option**. Extend to **DHCP helper** (**ip helper-address**) cross-subnet relays when using L3 gateways.

### 7.2 DNS & HTTP Servers

Hosts resolve names to lab IPs; browsers fetch simple pages—useful **Application layer** demos in **Simulation** mode.

### 7.3 Simple Email & FTP Lessons (Pedagogical)

Good for illustrating **plaintext risk** juxtaposed beside SSH/HTTPS narratives.

---

## Part 8 — NAT / PAT Story Arc

Pattern:

```
ip nat inside source list <ACL> interface <outside-if> overload
interface <inside-if>  ip nat inside
interface <outside-if> ip nat outside
```

Watchlist:

- Correct **inside/outside** marking.  
- ACL actually matches flows you think it matches (**extended ACL direction** misunderstandings bleed into NAT labs).

Verification:

```
show ip nat translations
show ip nat statistics
clear ip nat translation *
```

---

## Part 9 — ACLs With Intentionality

Draft ACLs as **traffic matrices**—**source**, **destination**, **protocol/port**, **action**, **interface direction**.

```
ip access-list extended …
permit …
deny …
```

Remember **implicit deny**. Place ACL on correct **in vs out** reference frame.

---

## Part 10 — Wireless (Version-Dependent Expectations)

PT sometimes includes **lightweight AP + WLC-style** teaching props—treat as **guided tutorial** more than feature-complete enterprise WLAN.

Document in your own notes: **SSID**, **security mode**, **management VLAN** vs **user VLAN** separation goal.

---

## Part 11 — Simulation Mode as a Teaching Superpower

In **Simulation:**

- Filter by **protocol** (ICMP, TCP, HTTP, etc.).  
- Step **event list** to see **encapsulation changes** at each hop.  
- Pair with **packet inspection** to narrate **DNS before TCP connect** before **HTTP GET**.

This builds the **storytelling** skill incident responders use.

---

## Part 12 — Documentation & Grading You Can Self-Apply

For every lab, produce:

1. **Topology diagram** (export image if available).
2. **Addressing table** (final, not draft).
3. **Config snippets** that are **non-default** only (cleaner review).
4. **Verification evidence** (command output highlights).
5. **Failure injection log**—one deliberate break + fix narrative weekly.

---

## Part 13 — Two Extensive Project Ideas

Use these as **capstone-style** builds. Adjust device models to what your Packet Tracer version offers.

---

### Project A — “Multi-Site Enterprise Core” (Routing + Switching + Services)

**Scenario:** A company has **Headquarters (HQ)** and **Branch A**. HQ has **two internal segments** (Staff VLAN 10, Servers VLAN 20). Branch A has **Users VLAN 30**. Internet is represented by a simplified **ISP router** with a loopback or stub network symbolizing public reachability.

**Learning objectives**

- Design **VLSM** from a single private block you choose (e.g., `10.X.0.0/16` carved intentionally).  
- Implement **802.1Q trunks** and **inter-VLAN routing** at HQ using **either** SVI on a multilayer switch **or** router-on-a-stick—**pick one** and justify in your write-up.  
- Connect HQ and Branch with **a routed WAN link** (Ethernet or serial per curriculum comfort).  
- Run **OSPF** (single area is fine; multi-area if you want stretch goals) so **all internal subnets** are reachable.  
- Place a **DHCP pool** for **at least one** client VLAN; use **helper addressing** if DHCP server lives on a different subnet.  
- Configure **PAT** at the edge so **internal hosts** can reach the **ISP stub** network.  
- Add **two ACL stories**:  
  - **Staff** may reach **Servers** on **HTTP only** (lab simplification), block other TCP to servers.  
  - **Branch users** may reach **Staff subnet** but **not** the **Server subnet** (explicit deny + logging mindset even if PT logging is limited).

**Minimum device list (flexible)**

- 2x multilayer-capable switch or 1 L3 switch + 1 access switch (adapt).  
- 2–3 routers (HQ edge, Branch router, ISP).  
- Several PCs + 1 server running **HTTP** (and **DHCP/DNS** if you consolidate services).

**Deliverables**

1. **Addressing & VLAN matrix** (one page).
2. **OSPF diagram** noting ABR-like behavior if you go multi-area.
3. **NAT/PAT boundary** diagram with inside/outside labels.
4. **Test matrix** (10+ rows): columns for **Test**, **Expected**, **Observed**, **Notes**.
5. **Troubleshooting appendix** with **three** intentional breaks you created (e.g., wrong native VLAN, OSPF passive on wrong interface, NAT ACL too narrow) and how you detected each.

**Stretch goals**

- Add **redundant trunk** between two access switches with **EtherChannel** (LACP where supported).  
- Introduce **basic HSRP** on the SVI path if your images support it cleanly—otherwise document why you skipped.  
- Export **PT activity** as a **.pkt** with a **README** listing device hostnames.

**Success criteria**

- Every PC gets **correct DHCP** (where designed) and **correct default gateway**.  
- **Inter-VLAN** works at HQ; **Branch ↔ Staff** works; **Branch ↔ Servers** fails per ACL story.  
- **PAT** translations appear when browsing HTTP to ISP-side content you host.  
- **Simulation mode** walkthrough narrating **DNS/ARP/ICMP** for one successful **cross-site ping**.

---

### Project B — “Security-Aware Mini Enterprise + Guest Network” (ACLs, Segmentation, Guest Isolation)

**Scenario:** A small business has **Internal Trusted**, **Guest Wi‑Fi**, and **Management** subnets. Wired internal users access file/HTTP services. Guests may only reach **internet** via PAT—**must not** access internal subnets. Management hosts may SSH to routers/switches **only from** management subnet IP ranges.

**Learning objectives**

- Practice **explicit deny** mentality—**least privilege by default**.  
- Implement **guest isolation** purely with **wired VLAN emulation** if wireless feature depth is insufficient (label the story as guest-equivalent VLAN).  
- Combine **extended ACLs**, **possibly standard ACL for VTY filtering** (topology-dependent)—know **management plane vs data plane**.  
- Use **SNMP or syslog placeholders** pedagogically—if unavailable, emulate with **scheduled show logging** screenshots after breaking ACLs responsibly.

**Suggested segmentation**


| VLAN | Role     | Routing policy summary                                                   |
| ---- | -------- | ------------------------------------------------------------------------ |
| 10   | INTERNAL | Reach servers + PAT internet                                             |
| 20   | SERVERS  | No default route to guests; return paths via firewall-like ACLs on L3    |
| 30   | GUEST    | PAT only; ACL blocks RFC1918 destinations except DNS resolver you expose |
| 99   | MGMT     | SSH to infrastructure; PCs in other VLANs cannot SSH                     |


**Technical tasks**

1. Build SVI gateways on a **L3 switch** or **central router**.
2. **Default route** posture: Internal may route broadly **under ACL governance**—Guests route **explicitly** with **narrow permits**.
3. Write ACLs readable by humans—**remark** lines if supported (`access-list … remark`).
4. Implement **management SSH**:
  - Create local users (`username … secret`).  
  - `line vty 0 4` plus `transport input ssh`.  
  - Generate **RSA key** modulus (PT often supports `**crypto key generate rsa`** pattern—if blocked, note limitation and workaround in doc).
5. **Defense-in-depth story:** even if a guest spoofed DNS, outline how **blocked private IP destinations** prevents lateral movement (**document** limitations of DNS exfil conceptual countermeasures—not full containment in PT).

**Deliverables**

1. **Threat model** half-page—three bullets **asset**, **adversary**, **control**.
2. **ACL linedoc** listing each ACL name, linked interfaces/directions, **plain-English intent**.
3. **Attack replay table**: five **mock attacks** (**guest pings internal server**, **guest browses permitted HTTP ISP stub**, **internal user SSH to router**, **mgmt SSH from wrong VLAN**, **internal blocked to management SSH**).

**Stretch goals**

- Add **IPv6 stubs** with **ICMPv6** neighbor discovery observation in Simulation—explain **dual-stack ACL pain** briefly.  
- Mirror **topology in Physical view** placing **HQ building** icons for storytelling polish.

**Success criteria**

- Guests reach **internet HTTP** lab page—**cannot** ping internal subnets.  
- Internal users **can** reach services—**blocked** appropriately per policy.  
- **SSH restriction** reproducibly enforced—provide **screenshots** or **verbatim CLI evidence**.

---

## Part 14 — Quality Check Before You Call a Lab Done

Ask:

1. If I **reload every device**, does the lab **recover** cleanly (startup-config saved)?
2. Can a **junior teammate** recreate from **your doc alone** inside one sitting?
3. Did I test **failure** at least once on purpose?

---

## Appendix A — Starter Command Cheat Sheet

```
enable
configure terminal
hostname R1
no ip domain-lookup
service password-encryption
banner motd ^
…
^
clock set …
copy running-config startup-config
write memory
```

**Interface quick path**

```
interface GigabitEthernet0/0
no shutdown
ip address …
description …
```

---

## Appendix B — Troubleshooting Mnemonic

**CDFR** adapted for beginners:

**Cabling → Duplex/IP → Filtering (ACL/NAT) → Routing**

Reorder if symptoms suggest **routing before ACL**—but **never assume routing** until L2+L3 basics check out.

---

## Appendix C — Integrity & Ethics Statement

Operate Packet Tracer topologies **as closed sandboxes**. Do not map offensive techniques taught elsewhere onto **production** or **third-party networks** without **authorization**.

---

### Closing

Packet Tracer rewards **iteration**. The two projects above deliberately mix **routing, switching, ACLs, NAT, DHCP, documentation, and deliberate failure**—the same rhythms as **Networking Academy capstones** and early **SOC/network engineering** onboarding.

Iterate, export your **.pkt**, and keep **change logs**: future-you will thank present-you during interviews when you narrate exact **verification** steps.