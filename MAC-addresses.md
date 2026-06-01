# MAC Addresses — Everything You Need to Know (CCNA)

**Audience:** CCNA and networking students who need to understand **Layer 2 addressing**, how switches forward traffic, and how MAC relates to IP.

**Related files:** [terminology.md](terminology.md) · [TCP_IP-model/TCP-IP-Model-Study-Guide.md](TCP_IP-model/TCP-IP-Model-Study-Guide.md) · [inter_vlan_routing.md](inter_vlan_routing.md) · [cheat-sheets/command-cheat-sheet.md](cheat-sheets/command-cheat-sheet.md)

---

## Part 1 — What Is a MAC Address?

A **MAC address** (Media Access Control address) is a **hardware identifier** used on **Layer 2** (Ethernet, Wi-Fi) to deliver a **frame** to the correct device on **one network segment** (one broadcast domain / VLAN).

| Property | Detail |
| -------- | ------ |
| Size | **48 bits** (6 bytes) |
| Written as | **12 hex digits**, usually in pairs: `AA:BB:CC:DD:EE:FF` |
| Scope | **Local** to a Layer 2 segment — routers do not forward MACs across IP subnets |
| Also called | **Burned-in address (BIA)**, **physical address**, **Ethernet address** |

**CCNA takeaway:** IP gets packets **across the network**. MAC gets frames **to the next hop** on the current link.

---

## Part 2 — MAC Address Format

### 2.1 Example breakdown

```text
MAC:  00:1A:2B:3C:4D:5E

      |---- OUI ----|  |-- NIC-specific --|
       00:1A:2B        3C:4D:5E
```

| Part | Bits | Meaning |
| ---- | ---- | ------- |
| **OUI** (first 24 bits / first 3 bytes) | 24 | **Organizationally Unique Identifier** — assigned to vendor (IEEE) |
| **Device ID** (last 24 bits) | 24 | Assigned by manufacturer (should be unique per device) |

### 2.2 Hex reminder

Each pair is one byte (8 bits). Example: `FF` = `11111111` = 255 decimal.

See [binary-hexadecimal/Binary-and-Hex-for-Networking.md](binary-hexadecimal/Binary-and-Hex-for-Networking.md) if hex is new.

### 2.3 Common separators

| Style | Example |
| ----- | ------- |
| Colon (most common) | `00:1A:2B:3C:4D:5E` |
| Dash (Windows sometimes) | `00-1A-2B-3C-4D-5E` |
| Cisco dotted | `001a.2b3c.4d5e` |

Same address, different display — exams may use any format.

---

## Part 3 — Special Bits in the First Octet (Exam Favorite)

The **second hex character** of the MAC (low nibble of first byte) tells you **unicast vs multicast** (and historically local vs global).

Look at the **first byte** in binary. The **least significant bit** (bit 0, rightmost):

| Bit 0 | Type |
| ----- | ---- |
| **0** | **Unicast** — one specific NIC |
| **1** | **Multicast** — group of devices |

**Example:** `01:00:5E:xx:xx:xx` — first byte `01` -> multicast (used in IPv4 multicast mapping).

The **second least significant bit** (bit 1) historically indicated:

| Bit 1 | Meaning |
| ----- | ------- |
| **0** | **Globally unique** (OUI assigned by IEEE) |
| **1** | **Locally administered** (LAA) — you or software set it |

**Exam trap:** Do not confuse **multicast MAC** with **broadcast**. Broadcast is the special all-1s address below.

---

## Part 4 — Broadcast and Multicast MAC Addresses

### 4.1 Broadcast MAC

```text
FF:FF:FF:FF:FF:FF
```

- Every device on the **VLAN** receives the frame (CPU processes if not filtered).
- Used for ARP, DHCP Discover, and other broadcasts.

### 4.2 Multicast MAC

- Starts with pattern where **I/G bit = 1** (multicast).
- IPv4 multicast maps to MAC range **`01:00:5E:00:00:00` – `01:00:5E:7F:FF:FF`** (simplified rule for CCNA).
- Switch may **flood** multicast unless IGMP snooping or similar is configured (awareness level).

### 4.3 Unicast MAC

- Normal one-to-one frame delivery on the LAN.
- Switch forwards only to the port where that MAC was learned (or floods if unknown).

---

## Part 5 — MAC vs IP Address

| | **MAC** | **IP** |
| - | ------- | ------ |
| Layer | 2 (Data link) | 3 (Network) |
| Length | 48 bits | 32 bits (IPv4) or 128 bits (IPv6) |
| Changes when? | Usually fixed per interface (unless spoofed/virtual) | Can change with DHCP, move subnets, VPN |
| Routed? | **No** — stays on same VLAN segment | **Yes** — routers forward by destination IP |
| Purpose | **Next hop** delivery on local link | **End-to-end** logical addressing |

### Same conversation uses both

```text
PC wants to ping 192.168.20.10 on another subnet:

1. PC knows dest IP is remote -> sends to default gateway IP
2. PC ARPs for gateway IP -> learns gateway MAC
3. Ethernet frame: dst MAC = gateway, src MAC = PC
4. Router routes by IP, builds NEW frame on outgoing VLAN
5. New frame has new src/dst MAC for that segment
```

**Rule:** MAC addresses **change at every router hop**. IP addresses (without NAT) typically **stay the same** source-to-destination.

---

## Part 6 — How Switches Use MAC Addresses

### 6.1 MAC address table (CAM table)

Switch **learns** source MAC on each port from incoming frames and **ages out** old entries.

```cisco
show mac address-table
```

| Column idea | Meaning |
| ----------- | ------- |
| VLAN | Which VLAN the MAC is in |
| MAC | Learned address |
| Type | Dynamic / Static / Secure |
| Port | Where to forward unicast to that MAC |

### 6.2 Forwarding decision

1. **Unicast to known MAC** -> forward out that port only.  
2. **Unicast to unknown MAC** -> **flood** out all ports in same VLAN except incoming port.  
3. **Broadcast / unknown multicast** -> flood within VLAN.

### 6.3 Aging

Entries time out (default often 300 seconds on Cisco). If host is silent, switch may forget MAC and flood again on next packet.

---

## Part 7 — ARP and MAC (IPv4)

**ARP** (Address Resolution Protocol) maps **IPv4 address -> MAC address** on the **local segment**.

```text
PC: "Who has 192.168.10.1? Tell 192.168.10.50"
     (ARP Request broadcast FF:FF:FF:FF:FF:FF)

Router: "192.168.10.1 is at MAC 00:11:22:33:44:55"
          (ARP Reply unicast)
```

| Command (host/router) | Purpose |
| --------------------- | ------- |
| `show ip arp` | IPv4 to MAC bindings on Cisco router |

**Gratuitous ARP:** Host announces its own IP/MAC — used after IP change or redundancy (HSRP virtual MAC awareness).

**IPv6:** Uses **NDP** (Neighbor Discovery) instead of ARP — same idea, different protocol.

---

## Part 8 — Static, Dynamic, and Secure MAC Entries

| Type | How it gets in the table | CCNA note |
| ---- | ------------------------ | --------- |
| **Dynamic** | Learned from traffic | Default behavior |
| **Static** | Admin configured (`mac address-table static`) | Pin MAC to port |
| **Sticky** | Learned once, then saved like static | Used with **port security** |

### Port security (brief)

Limits which MAC(s) may appear on an access port; violation can **shutdown** port.

```cisco
switchport port-security
switchport port-security maximum 1
switchport port-security mac-address sticky
```

---

## Part 9 — Virtual and Redundant MAC Addresses

| Use case | Example |
| -------- | ------- |
| **Virtual NIC** | VM, container, hypervisor assigns MAC |
| **HSRP / VRRP / GLBP** | Virtual **gateway MAC** shared by routers |
| **MAC spoofing** | Attacker fakes source MAC (security risk) |
| **Locally administered** | Bit set when admin/software overrides burned-in address |

**CCNA:** Know that a **virtual IP** for FHRP often has a **virtual MAC** hosts use as default gateway L2 target.

---

## Part 10 — MAC Addresses in Wireless

| Term | MAC role |
| ---- | -------- |
| **BSSID** | Usually the **radio MAC** of the AP (looks like a MAC address) |
| **Client MAC** | Phone/laptop wireless interface |
| **SSID** | Network **name** — not a MAC (do not confuse) |

Wireless frames still bridge into wired Ethernet with MAC addresses.

---

## Part 11 — Troubleshooting with MAC in Mind

| Symptom | MAC-layer things to check |
| ------- | ------------------------- |
| Hosts same VLAN cannot ping | Wrong VLAN, port down, MAC not learned, storm |
| Can ping IP but intermittent | MAC flapping, duplicate MAC, aging |
| Works until you move cable | Sticky/static MAC on wrong port |
| Only one device works on port | Port security violation (err-disabled) |

### Useful commands (Cisco switch)

```cisco
show mac address-table
show mac address-table dynamic
show mac address-table address xxxx.xxxx.xxxx
show port-security
show interfaces status
```

### Clear MAC table (lab only — disruptive)

```cisco
clear mac address-table dynamic
```

---

## Part 12 — Common CCNA Exam Traps

| Wrong idea | Correct idea |
| ---------- | ------------ |
| "Routers forward by MAC to distant networks" | Routers forward by **IP**; rewrite **MAC** each hop |
| "MAC and IP are interchangeable" | They work **together** at different layers |
| "Switches route by IP" | Basic switch forwards by **MAC** (unless L3 switch with SVI) |
| "FF:FF:FF:FF:FF:FF is multicast" | It is **broadcast** |
| "MAC never changes" | Changes **per router hop**; may change with VM/FHRP/spoofing |

---

## Part 13 — Practice Questions (Self-Check)

1. How many bits in a standard Ethernet MAC address?  
2. What does `FF:FF:FF:FF:FF:FF` represent?  
3. Why does a PC ARP its default gateway when pinging a remote server?  
4. Does the destination MAC stay the same end-to-end across two routers?  
5. What command shows the switch MAC table?  
6. What is an OUI?

### Answers

1. **48 bits** (6 bytes).  
2. **Layer 2 broadcast** — all devices on the VLAN receive the frame.  
3. Remote IP is off-subnet; L2 delivery must target the **gateway's MAC**.  
4. **No** — new L2 header on each routed segment.  
5. **`show mac address-table`**.  
6. First **24 bits** identifying the **manufacturer** (IEEE assignment).

---

## Part 14 — Quick Reference Card

```text
MAC = 48-bit Layer 2 address (hex, 6 bytes)
OUI = first 3 bytes (vendor)
Unicast = I/G bit 0 in first octet
Multicast = I/G bit 1
Broadcast = FF:FF:FF:FF:FF:FF
Switch learns src MAC -> CAM table
Unknown unicast -> flood
ARP = IP to MAC on local segment
Router hop -> new src/dst MAC, same IP (typical)
```

---

**Mastery check:** In Packet Tracer Simulation mode, ping across a router and watch **MAC addresses change** on each link while **IP stays the same** — that single observation ties most of this file together.
