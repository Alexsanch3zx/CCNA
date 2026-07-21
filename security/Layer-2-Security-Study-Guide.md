# Layer 2 Security — DHCP Snooping, DAI, IP Source Guard

**Audience:** CCNA students who need to understand **switch features that stop rogue DHCP, ARP spoofing, and IP/MAC spoofing** on the access layer.

**Folder:** `security/` — network and device security theory (pairs with [Device-Hardening-Study-Guide.md](Device-Hardening-Study-Guide.md) and [../ACL-Study-Guide.md](../ACL-Study-Guide.md)).

**Related files:** [../terminology.md](../terminology.md) · [../MAC-addresses.md](../MAC-addresses.md) · [../IP-addresses/DHCP_Notes.md](../IP-addresses/DHCP_Notes.md) · [../cheat-sheets/command-cheat-sheet.md](../cheat-sheets/command-cheat-sheet.md) · [../labs/CCNA_Small_Office_Network_Lab.md](../labs/CCNA_Small_Office_Network_Lab.md) · **Hands-on lab:** [../labs/Layer2_Security_DHCP_Snooping_DAI/CCNA_Layer2_Security_Lab.md](../labs/Layer2_Security_DHCP_Snooping_DAI/CCNA_Layer2_Security_Lab.md)

---

## Part 1 — Why Layer 2 Security Matters

Many attacks happen **inside the LAN**, below routers and firewalls:

| Attack | What the attacker does |
| ------ | ---------------------- |
| **Rogue DHCP** | Answers DHCP Discover with a bad gateway/DNS |
| **ARP spoofing / poisoning** | Claims to be the gateway’s MAC |
| **IP spoofing** | Uses someone else’s IP on an access port |

**Port security** (sticky MAC / violation shutdown) helps with unauthorized **MACs**. The trio below adds protection around **DHCP and ARP/IP bindings**:

1. **DHCP Snooping**
2. **Dynamic ARP Inspection (DAI)**
3. **IP Source Guard (IPSG)**

**CCNA takeaway:** These features work as a **set**. DHCP snooping builds the binding table; DAI and IPSG **consume** it.

---

## Part 2 — Trusted vs Untrusted Ports

All three features use the same mental model:

| Port type | Typical connection | Treat as |
| --------- | ------------------ | -------- |
| **Trusted** | Uplink to distribution, port toward **real DHCP server**, inter-switch links | Legitimate control-plane DHCP/ARP from infrastructure |
| **Untrusted** | Access ports to PCs / users | Strict validation |

**Rule of thumb:** End-user access ports = **untrusted**. DHCP server / uplink path = **trusted**.

Mis-trusting a user port (marking it trusted) defeats the feature.

---

## Part 3 — DHCP Snooping

### 3.1 Problem

A rogue device runs DHCP and gives clients:

- Wrong **default gateway** (attacker machine)
- Wrong **DNS** (phishing)

Clients happily configure the bad settings (DORA still “works”).

### 3.2 What DHCP snooping does

On **untrusted** ports, the switch:

- Allows client DHCP messages (**Discover / Request**) as appropriate
- **Drops** DHCP **server** messages (**Offer / Ack**) from untrusted ports
- Builds a **DHCP snooping binding table**: MAC ↔ IP ↔ VLAN ↔ port ↔ lease

On **trusted** ports, server messages are allowed.

### 3.3 Conceptual config

```cisco
ip dhcp snooping
ip dhcp snooping vlan 10,20

interface fa0/1
 switchport mode access
 switchport access vlan 10
 ip dhcp snooping trust          ! only on real server/uplink path

interface fa0/10
 switchport mode access
 switchport access vlan 10
 ! no trust = untrusted access port
```

Optional rate-limit (awareness): limit DHCP messages on untrusted ports to reduce floods.

### 3.4 Verify

```cisco
show ip dhcp snooping
show ip dhcp snooping binding
```

**Binding table** is the gold — DAI and IPSG need it.

### 3.5 Option 82 (awareness)

Switches may insert **DHCP Option 82** relay info. Some servers reject it unless configured — a common lab “DHCP broken after enabling snooping” footgun. Know the symptom; details vary by platform.

---

## Part 4 — Dynamic ARP Inspection (DAI)

### 4.1 Problem

Attacker sends forged ARP replies: “I am `192.168.10.1`” (the gateway) with the attacker’s MAC. Victims send traffic to the attacker (**man-in-the-middle**).

### 4.2 What DAI does

DAI inspects **ARP** requests/replies on **untrusted** ports and checks them against the **DHCP snooping binding table** (and optional static ARP ACLs).

| ARP claim | Result |
| --------- | ------ |
| IP/MAC matches a valid binding for that port | **Allow** |
| IP/MAC does not match | **Drop** (and can log) |

Trusted ports skip inspection (infrastructure).

### 4.3 Dependency

**DAI without DHCP snooping (or ARP ACLs) is incomplete** for dynamic clients — there is nothing trustworthy to validate against.

### 4.4 Conceptual config

```cisco
ip arp inspection vlan 10,20

interface fa0/24
 ip dhcp snooping trust
 ip arp inspection trust          ! uplink / toward gateway
```

Access ports: leave **untrusted** (default when DAI enabled for the VLAN).

### 4.5 Verify

```cisco
show ip arp inspection
show ip arp inspection statistics
show ip dhcp snooping binding
```

---

## Part 5 — IP Source Guard (IPSG)

### 5.1 Problem

A host configures a **static IP** that belongs to someone else, or spoofs a source IP to bypass simple filters.

### 5.2 What IPSG does

On an untrusted access port, the switch permits traffic only if the **source IP** (and often **MAC**) matches the DHCP snooping binding for that port.

Typically implemented with a dynamic **port ACL / VACL-like filter** built from bindings.

### 5.3 Modes (conceptual)

| Mode | Checks |
| ---- | ------ |
| **Source IP** | Source IP must match binding |
| **Source IP + MAC** | Tighter — IP and MAC must match |

### 5.4 Conceptual config

```cisco
interface fa0/10
 switchport mode access
 ip verify source                ! IP only (syntax varies)
 ! or:
 ip verify source port-security  ! IP + MAC style on many platforms
```

**Static hosts** (printers, servers) need a **static binding** if they do not use DHCP — otherwise IPSG can black-hole them.

### 5.5 Verify

```cisco
show ip verify source
show ip dhcp snooping binding
```

---

## Part 6 — How the Three Fit Together

```text
DHCP Snooping
   │
   ├─ blocks rogue DHCP Offers on untrusted ports
   │
   └─ builds BINDING TABLE (MAC, IP, VLAN, port)
            │
            ├──► DAI  — validate ARP against bindings
            └──► IPSG — validate data-plane source IP(/MAC) against bindings
```

| Feature | Plane it mostly protects | Spoof it stops |
| ------- | ------------------------ | -------------- |
| DHCP snooping | DHCP control | Rogue gateway/DNS via DHCP |
| DAI | ARP | Gateway/host ARP poisoning |
| IPSG | IP data plane on the port | Source IP spoof from that port |

**Port security** still matters for limiting **how many MACs** and sticky learning — complementary, not a replacement.

---

## Part 7 — Enable Order (Safe Mental Sequence)

1. Know where the **real DHCP server** is
2. Enable **DHCP snooping** globally + per VLAN
3. Mark **trusted** ports (server path / uplinks)
4. Confirm clients still get leases → check **bindings**
5. Enable **DAI** on those VLANs; trust the same infrastructure ports
6. Enable **IPSG** on access ports; add static bindings for non-DHCP devices

Turning everything on at once without trust ports = **everyone loses DHCP/ARP** and the LAN looks “down.”

---

## Part 8 — Common Mistakes & Exam Traps

| Wrong idea | Correct idea |
| ---------- | ------------ |
| “Trust all access ports so users work” | Access ports should stay **untrusted** |
| “DAI replaces DHCP snooping” | DAI **uses** snooping bindings |
| “IPSG is a firewall ACL on the router” | IPSG is a **switch port** source check |
| “Static servers work automatically with IPSG” | Need **static bindings** or DHCP |
| “These are Layer 3 router features only” | Classic deployment is on **access/distribution switches** |

---

## Part 9 — Practice Questions (Self-Check)

1. What DHCP messages should be dropped on an untrusted port?
2. What table does DAI use to validate ARP?
3. What does IP Source Guard check on a port?
4. Should the port facing a legitimate DHCP server be trusted or untrusted for snooping?
5. Name one attack DHCP snooping is meant to stop.

### Answers

1. DHCP **server** messages (**Offer / Ack**, etc.) from untrusted ports.
2. The **DHCP snooping binding** table (and optional ARP ACLs).
3. That the **source IP** (and optionally MAC) matches the binding for that port.
4. **Trusted**.
5. **Rogue DHCP** (malicious gateway/DNS assignment).

---

## Part 10 — Quick Reference Card

```text
Untrusted = user access ports
Trusted   = DHCP server path / uplinks

DHCP Snooping: drop rogue Offers; build bindings
DAI:          check ARP vs bindings
IPSG:         check src IP(/MAC) vs bindings

Order: snooping + trust → bindings → DAI → IPSG
Static hosts need static bindings for IPSG

Verify:
  show ip dhcp snooping binding
  show ip arp inspection
  show ip verify source
```

---

**Mastery check:** Explain in one paragraph why enabling DAI before DHCP snooping (with no ARP ACLs) is a bad idea — if you can, you understand the dependency.
