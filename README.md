# 6CCNA Study Guide & Learning Journal

Personal repository for **Cisco CCNA (200-301)** prep, Packet Tracer labs, and notes as I move into tech.

**This README is my reflection journal** — I add entries over time about what I actually understood, what confused me, and what I still need to practice.

---

## How I Use the Reflection Section

When I finish a chapter, lab, or study session, I add an entry below. I try to answer:

1. **What did I learn?**
2. **What clicked?**
3. **What is still fuzzy?**
4. **What will I lab or review next?**

Short entries are fine. Consistency matters more than length.

### Copy-paste template for a new entry

```markdown
### YYYY-MM-DD — [Topic or lab name]

**What I learned:**

**What clicked:**

**Still fuzzy:**

**Next steps:**
```

---

## Reflections

### 2026-07-21 — Hub vs switch

**What I learned:** A **hub** is a Layer 1 multiport repeater — it takes bits in on one port and **repeats them out every other port**. Everyone shares one big **collision domain**, so only one device should talk at a time (half-duplex / CSMA/CD thinking). A **switch** is Layer 2 — it reads **MAC addresses**, builds a **MAC address table**, and **forwards** a frame only toward the destination port when it knows where that MAC lives (flooding only for unknown/broadcast). Each switch port is typically its **own collision domain**, so full duplex is normal.

**What clicked:** “Hub = dumb copy to all; switch = smart forward by MAC” is the exam-friendly version. That also explains why modern LANs use switches: less wasted bandwidth, fewer collisions, and features like VLANs, STP, and port security only make sense once you’re filtering by frame/MAC instead of blasting every port. Linking this to my MAC-table notes: the table is exactly what a hub does **not** have.

**Still fuzzy:** When people say “broadcast domain” for a hub vs an unmanaged switch — both flood broadcasts by default; VLANs (or a router) are what split broadcast domains, not “switch vs hub” alone.

**Next steps:** Skim [MAC-addresses.md](MAC-addresses.md) and [networking-basics.md](networking-basics.md); in Packet Tracer, contrast a hub topology (everyone sees every frame in Simulation) with a switch that unicast-forwards after learning.

---

### 2026-07-21 — Wireless principles (BSS, ESS, SSID, roaming)

**What I learned:** Wireless LANs still end up on a **wired** switch/router — the AP is a bridge between radio and Ethernet. A **BSS** is one AP and its clients (one cell). An **ESS** is two or more BSSs that share the **same SSID**, same security, and the same LAN/VLAN so a client can **roam** between APs and keep the same IP and gateway. The **SSID** is the network name people see; the **BSSID** is usually the AP radio’s MAC and identifies *which* cell you’re on. Non-overlapping channels (often **1, 6, 11** in 2.4 GHz) matter so neighboring APs don’t interfere.

**What clicked:** In the [ESS Wireless LAN lab](labs/ESS_Wireless_LAN/CCNA_ESS_Wireless_Lab.md), two APs with identical `Campus-WiFi` settings felt like one WLAN, not two networks. Roaming kept working because both APs sat on the same switch VLAN and got DHCP from the same router pool. Typo the SSID on one AP and you suddenly have two networks — that made ESS vs “two separate WLANs” obvious.

**Still fuzzy:** Enterprise wireless (controllers, CAPWAP, WPA2/WPA3-Enterprise with RADIUS) vs the autonomous AP-PT model in Packet Tracer — I know the names more than the packet flow.

**Next steps:** Re-run the ESS lab roaming check; review wireless terms in [terminology.md](terminology.md); practice channel planning and “associates but no IP” troubleshooting (DHCP vs AP vs SSID mismatch).

---

### 2026-07-21 — Port security (sticky MAC and violation shutdown)

**What I learned:** **Port security** limits which **MAC addresses** are allowed on a switch access port. Typical lab pattern: `switchport port-security`, `maximum 1`, `mac-address sticky` (learn and remember the first host’s MAC), and `violation shutdown`. If a second device plugs into that port and sends traffic, the switch treats it as a violation. With **shutdown** mode the port goes **err-disabled** / **Secure-shutdown** until you recover it (`shutdown` then `no shutdown`). Softer modes like **restrict** or **protect** drop unauthorized frames (and may count violations) without killing the whole port.

**What clicked:** Port security is about **how many / which MACs** on a port — not the same job as DHCP snooping or DAI. Sticky learning made sense after I generated traffic from the legit PC first (`ping` the gateway), then checked `show port-security interface fa0/1` for **Secure-up**, sticky count **1**, and violation count **0**. Swapping in another device flipped it to **Secure-shutdown** — that was the “unauthorized laptop on my desk drop” moment.

**Still fuzzy:** Whether sticky MACs survive a reload without saving them into the running/startup config the way I expect in Packet Tracer vs real IOS.

**Next steps:** Finish the port-security violation demo in the [Layer 2 Security Lab](labs/Layer2_Security_DHCP_Snooping_DAI/CCNA_Layer2_Security_Lab.md); compare with the earlier pass in the [Small Office Network Lab](labs/CCNA_Small_Office_Network_Lab.md); practice `show port-security address` after sticky learning.

---

### 2026-07-21 — MAC address tables and Spanning Tree Protocol (STP)

**What I learned:** A switch builds a **MAC address table** by learning the **source MAC** on each frame and the port it arrived on. Later, if the **destination MAC** is in the table, the switch **forwards** out only that port; if unknown (or broadcast), it **floods**. That flooding is why redundant switch links without loop control are dangerous — frames can circle forever. **Spanning Tree Protocol (STP)** stops Layer 2 loops by electing a **root bridge** and putting some ports in a **blocking** (or alternate) state so only one active path exists in the logical tree, while physical redundancy stays for failover.

**What clicked:** MAC learning and STP solve different halves of the same story. The MAC table answers “where is this host?” STP answers “which links are allowed to forward so floods don’t storm?” Without STP, MAC tables also go unstable (**flapping** — the same MAC learned on different ports as looped frames bounce around). Commands that made this concrete: `show mac address-table` for learning/forwarding, and `show spanning-tree` for root, roles (root / designated / blocked), and which VLAN’s tree I’m looking at (Rapid PVST+ is per VLAN).

**Still fuzzy:** Quickly picking **root port vs designated port** on a diagram under time pressure, and when to use `spanning-tree vlan X root primary` vs setting priority by hand.

**Next steps:** Re-read [MAC-addresses.md](MAC-addresses.md) and [STP-Spanning-Tree-Study-Guide.md](STP-Spanning-Tree-Study-Guide.md); in the [Layer 3 Switching / EtherChannel lab](labs/CCNA_Layer3_Switching_Redundancy_Lab.md), force a root and watch a port block, then check the MAC table before and after breaking a link.

---

### 2026-07-21 — Packet Tracer: PCs cannot run a DHCP server (Layer 2 Security Lab)

**What I learned:** While building the rogue DHCP demo in the [Layer 2 Security Lab](labs/Layer2_Security_DHCP_Snooping_DAI/CCNA_Layer2_Security_Lab.md), I found that Packet Tracer **PC** devices do **not** support running a DHCP **server**. A PC can be a DHCP **client** (Desktop → IP Configuration → DHCP), but there is no **Services → DHCP** server to turn on like you might expect on a real compromised host.

**What clicked:** For rogue-DHCP labs in PT, use a **Server-PT** instead (End Devices → Server). Give it a static IP (e.g. `192.168.10.50`), open **Services → DHCP**, turn the service **On**, and set a bad default gateway so clients that lease from it get redirected. Legitimate DHCP still comes from the router (or a trusted server); the Server-PT is only the attacker on an untrusted access port.

**Still fuzzy:** How closely PT’s Server DHCP matches real IOS/`dhcpd` behavior (Option 82, race with multiple Offers) when testing snooping edge cases.

**Next steps:** Finish the snooping “before vs after” demo with **ROGUE** as Server-PT; confirm PC0 keeps gateway `192.168.10.1` once Fa0/2 is untrusted.

---

### 2026-06-04 — Access Control List (ACL) on a subinterface = Router-on-a-Stick (Small Office Lab)

**What I learned:** In the [Small Office Network Lab](labs/CCNA_Small_Office_Network_Lab.md), putting an ACL on `**interface g0/0.20`** only works because **inter-VLAN routing** is already built with **router-on-a-stick**. One physical port (`g0/0`) trunks to the switch; each VLAN gets a **subinterface** (`g0/0.10`, `g0/0.20`, …) with `**encapsulation dot1Q`** and a **gateway IP** (e.g. `192.168.20.1` for Sales). PCs in different VLANs cannot talk at Layer 2 — they send to their **default gateway**, and the router routes between subnets.

**What clicked:** Applying the ACL to `**g0/0.20 in`** means traffic **from Sales entering the router** is filtered before it can reach HR or other VLANs. Router-on-a-stick makes the router the **only path** between VLANs, so ACLs on subinterfaces are a logical place to enforce policy.

**Still fuzzy:** Picking `**in` vs `out`** on the first try without drawing which way the packet flows.

**Next steps:** Test Sales → HR (blocked) vs HR → Sales (allowed with my lab ACL); read [inter_vlan_routing.md](inter_vlan_routing.md).

---

### 2026-06-04 — Encapsulation (data going down the stack)

**What I learned:** **Encapsulation** is how data gets wrapped layer by layer **on the way out**. The application sends data; each lower layer adds its own header (TCP/UDP, then IP, then Ethernet frame with MAC addresses, then bits on the wire). On the way **in**, the device **de-encapsulates** — strips headers from the bottom up. A **switch** cares about the **frame (MAC)**; a **router** looks at the **packet (IP)** and builds a **new frame** for the next hop.

**What clicked:** When I ping another PC on the same VLAN, the IP addresses stay the same on that hop, but if the packet crosses a **router**, the **MAC addresses change** every time — IP usually does not (without NAT). Packet Tracer **Simulation** mode lets me watch this happen step by step.

**Still fuzzy:** Remembering the exact **PDU names** (segment vs packet vs frame) without mixing them up on exam questions.

**Next steps:** Trace one **same-VLAN** ping and one **cross-VLAN** ping in Simulation; review [TCP-IP-Model-Study-Guide.md](TCP_IP-model/TCP-IP-Model-Study-Guide.md) Part 4.

---

### 2026-06-03 — Checking VLANs (CLI and GUI)

**What I learned:** I can verify VLANs with `show vlan brief` in the switch CLI, or click the switch in Packet Tracer → **Config** → **VLAN Database** to see the VLANs I created without typing commands.

**What clicked:** CLI and GUI show the same info — good for lab checks when I forget a command.

---

### 2026-06-03 — IPv4 is 32 bits and `/24` math (`2^n` and `2^n - 2`)

**What I learned:** Every **IPv4 address is 32 bits** total, written as four octets (e.g. `192.168.10.0`). The `**/24`** in `192.168.10.0/24` means **24 bits are network** and the rest are **host** bits. So: **32 - 24 = 8 host bits**. From there:


| Calculation                         | Formula               | For `/24` (8 host bits) |
| ----------------------------------- | --------------------- | ----------------------- |
| Total addresses in the subnet       | `2^n` (n = host bits) | `2^8` = **256**         |
| Usable host addresses (typical LAN) | `2^n - 2`             | `2^8 - 2` = **254**     |


The **-2** is because the **network address** (host bits all 0) and **broadcast** (host bits all 1) are not assigned to normal hosts.

**What clicked:** The slash number is not “how many hosts” — it is **how many network bits**. I subtract from **32** first, then use **n** in `2^n`.

**Still fuzzy:** Doing the same math when the subnet boundary is not in the last octet (e.g. `/22` or `/26) — I need to know which octet the block size lands in.

**Next steps:** Practice three random prefixes (`/26`, `/27`, `/30`) on paper; use [subnetting.md](IP-addresses/subnetting.md) for VLSM next.

---

### 2026-06-02 — DHCP (Dynamic Host Configuration Protocol)

**What I learned:** **DHCP** assigns IP settings automatically (address, mask, default gateway, DNS) so I do not have to type them on every PC. The client process is **DORA**: **D**iscover, **O**ffer, **R**equest, **A**cknowledge. On the router in my lab, I create **pools** per VLAN (e.g. HR `192.168.10.0`) and set **default-router** to the subinterface IP (`.1`). `**ip dhcp excluded-address`** keeps specific IPs free for static devices (server, printer). PCs in Packet Tracer use **DHCP** in Desktop → IP Configuration instead of static.

**What clicked:** The **default gateway** in the pool must be the router interface for that VLAN — otherwise PCs get an IP but cannot reach other subnets. DHCP is an **Application layer** service, but it only works if **L2/L3** (VLANs, routing, trunks) are already correct.

**Still fuzzy:** When the DHCP server is on a **different subnet** than the clients — I need `**ip helper-address`** on the router to relay broadcasts (not needed when R1 is the server and default gateway for each VLAN in my small office lab).

**Next steps:** Run `show ip dhcp binding` on R1 after PCs lease; re-read [DHCP_Notes.md](IP-addresses/DHCP_Notes.md); finish DHCP step in [Small Office Network Lab](labs/CCNA_Small_Office_Network_Lab.md).

---

### 2026-06-02 — What `Fa0/23` means

**What I learned:** On a Cisco switch, `**Fa0/23`** is interface notation: `**Fa`** = FastEthernet, `**0`** = module/slot (on a 2960-style switch this is usually fixed at 0), `**23**` = port number. So it is **port 23** on the FastEthernet module — not “port 0 and port 23.” In the [Small Office Network Lab](labs/CCNA_Small_Office_Network_Lab.md), **SW1 Fa0/23** connects to **SW2 Fa0/23** as the **trunk** between switches.

**What clicked:** The slash separates **where** on the device (slot) from **which port** on that module.

**Still fuzzy:** When labs use `Gig0/0` on a router vs `Fa0/24` on a switch — same idea, different interface type prefix.

**Next steps:** Match port labels on the topology diagram to `interface fa0/23` in config without mixing up access ports (PCs) and trunk ports.

---

### 2026-06-01 — What `VLANs` are

### 2026-01-01 — First VLAN lab

**What I learned:** VLANs split one switch into separate broadcast domains. Access ports belong to one VLAN; trunks carry multiple VLANs with 802.1Q tags.

**What clicked:** Without a router or L3 switch, VLAN 10 and VLAN 20 cannot talk to each other — same switch, different L2 domains.

**Still fuzzy:** Native VLAN mismatches on trunks and when I need `switchport trunk native vlan` vs leaving default.

**Next steps:** Finish [Small Office Network Lab](labs/CCNA_Small_Office_Network_Lab.md); practice `show vlan brief` and `show interfaces trunk`.

---

*Last updated: 6/6*