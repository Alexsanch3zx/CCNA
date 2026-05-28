# Cisco CCNA–Aligned Networking Study Guide

### Associate-Level Depth with Advanced Protocol & Design Treatment

**Audience:** Career switchers entering tech who want rigorous preparation for **Cisco Certified Network Associate (CCNA)** — Exam **200-301**.

This guide aligns topic coverage and depth with the official **CCNA 200-301** blueprint (six domains). Use it as a structured syllabus; pair it with hands-on labs (Cisco Packet Tracer, Cisco Modeling Labs, or physical gear) and Cisco’s official curriculum where possible.

---

## How to Use This Guide

1. **Read in order** for the first pass (fundamentals → access → routing → services → security → automation).
2. **Revisit by domain** when you are two to four weeks from your exam date.
3. **For every major topic**, be able to explain: *what problem it solves*, *how it works at packet/frame level where relevant*, *how to verify*, *how it fails*, *how to troubleshoot*.
4. **Lab everything** that has a CLI: switching, routing, OSPF, NAT, ACLs, DHCP, basic automation APIs.

---

## Part 0 — Certification Context & Exam Strategy

### CCNA (200-301) at a Glance


| Attribute         | Detail                                                                                                                  |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Exam code         | **200-301 CCNA**                                                                                                        |
| Format            | Multiple choice, drag-and-drop, simlets; typically ~100–120 questions, ~120 minutes                                     |
| Prerequisite      | None (single-exam associate certification)                                                                              |
| Validity          | Certification valid three years; renewal via exams or Continuing Education                                              |
| Blueprint domains | Network Fundamentals; Network Access; IP Connectivity; IP Services; Security Fundamentals; Automation & Programmability |


### Skills Employers Pair With CCNA

- **L2/L3 support**: VLANs, trunking, routing, basic wireless and security.  
- **Operations mindset**: change windows, documentation, troubleshooting methodology.  
- **Growth path**: **CCNP Enterprise** (professional), **DevNet Associate**, cloud/network automation roles.

### Study Architecture (12–16 Week Example)


| Weeks | Focus                                                         |
| ----- | ------------------------------------------------------------- |
| 1–2   | Models, encapsulation, cabling, Ethernet, IPv4 math           |
| 3–5   | Switching: VLANs, STP, EtherChannel, basic troubleshooting    |
| 6–9   | IPv4/IPv6, static routing, OSPF single-area                   |
| 10–11 | DHCP, DNS, NAT/PAT, NTP, SNMP/syslog basics                   |
| 12    | ACLs, port security, wireless and security concepts           |
| 13–14 | Automation: APIs, JSON, Python/`requests` or Ansible concepts |
| 15–16 | Full practice exams, weak-area drills, timed simlets          |


Adjust pacing if you have daily lab time versus weekends-only.

---

## Domain 1 — Network Fundamentals (~20%)

### 1.1 OSI and TCP/IP Models (Conceptual Rigor)

**OSI (reference)** — seven layers; memorize functions and **PDU names**:


| Layer | Name         | PDU     | Role (concise)                                        |
| ----- | ------------ | ------- | ----------------------------------------------------- |
| 7     | Application  | Data    | User-facing protocols (HTTP, SMTP, etc.)              |
| 6     | Presentation | Data    | Encoding, encryption (often collapsed into app stack) |
| 5     | Session      | Data    | Dialog control (often collapsed)                      |
| 4     | Transport    | Segment | End-to-end reliability, multiplexing (TCP/UDP ports)  |
| 3     | Network      | Packet  | Logical addressing, routing (IPv4/IPv6)               |
| 2     | Data Link    | Frame   | Hop-to-hop delivery on LAN/WAN link (MAC, Ethernet)   |
| 1     | Physical     | Bits    | Signaling, media, connectors                          |


**TCP/IP model (practical)** — four layers mapped to OSI:

- **Application** (OSI 5–7): HTTP, HTTPS, DNS, DHCP, SSH, SNMP.  
- **Transport** (OSI 4): TCP (connection-oriented, reordering, windowing), UDP (connectionless, low overhead).  
- **Internet** (OSI 3): IPv4, IPv6, ICMP, routing protocols (OSPF runs “above” IP but is control-plane).  
- **Network Access** (OSI 1–2): Ethernet, Wi‑Fi, PPP.

**Encapsulation**: At each layer down, headers (and sometimes trailers, e.g., FCS at L2) wrap the payload. **De-encapsulation** occurs at the receiver. Routers process up to L3; switches typically L2 (unless SVI/routed ports).

### 1.2 Network Types & Characteristics

- **LAN vs WAN vs MAN** — ownership, geographic scope, typical technologies (Ethernet LAN; MPLS/Internet VPN/SDWAN WAN).  
- **Topologies**: physical vs logical (e.g., physical star with logical bus in legacy shared Ethernet).  
- **Client-server vs peer-to-peer**.  
- **Point-to-point vs multipoint** (critical for WAN and wireless).

### 1.3 Cabling & Physical Standards

- **Copper**: Cat5e/Cat6/Cat6a — **100 m** channel limit for Ethernet over twisted pair; **straight-through** vs **crossover** (modern NIC auto-MDIX reduces pain).  
- **Fiber**: SM vs MM; optics (SR/LR); cleanliness and loss budgets (conceptual for CCNA).  
- **PoE**: power classes; **802.3af/at/bt** awareness — VoIP APs and cameras.

### 1.4 Ethernet Fundamentals (Advanced Treatment)

- **Frames**: preamble/SFD, DA/SA, EtherType/Length, payload, FCS.  
- **MAC addressing**: unicast, multicast, broadcast; **OUI**; locally administered bits.  
- **Duplex/speed**: autonegotiation failure modes (duplex mismatch legacy issue).  
- **Collision domains vs broadcast domains** — **each switch port** is its own collision domain in full-duplex switched Ethernet; **routers** (and VLAN boundaries) segment broadcast domains.

### 1.5 IPv4 Addressing & Subnetting (Non‑Negotiable Fluency)

**Core skills:**

- Convert decimal ↔ binary quickly for octets you manipulate often.  
- Given **network/prefix**, list network address, broadcast, first/last host, **subnet ID**, usable hosts 2^{(32-p)} - 2 for typical subnets.  
- **VLSM**: subnet a given block to minimize waste while satisfying host counts per segment.  
- **Route summarization / supernetting**: aggregate contiguous prefixes where policies allow.

**Private RFC1918**: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`.

**Special-use awareness**: loopback `127.0.0.0/8`, link-local APIPA `169.254.0.0/16`, multicast class D, limited broadcast.

### 1.6 IPv6 Essentials

- **128-bit** representation; **compression rules** (leading zeros, `::` once).  
- **Global unicast**, **unique local** (`fc00::/7`), **link-local** (`fe80::/10` — mandatory on every interface).  
- **NDP** replaces ARP: **NS/NA**, **RS/RA**, **SLAAC** vs **DHCPv6**.  
- **EUI-64** vs privacy extensions (conceptual).

### 1.7 TCP & UDP (Transport Layer Depth)

- **TCP three-way handshake** (SYN, SYN-ACK, ACK); **windowing**, **reliability**, **ordered delivery**, **flow control**.  
- **Connection teardown** (FIN/ACK exchange).  
- **UDP**: DNS often, VoIP/video real-time, low latency; no guarantee.

**Well-known ports** (sample): 20/21 FTP, 22 SSH, 23 Telnet, 25 SMTP, 53 DNS (TCP/UDP), 67/68 DHCP, 80 HTTP, 443 HTTPS, 161 SNMP.

### 1.8 Network Troubleshooting Methodology

Structured flow (Cisco-friendly):

1. **Identify** the problem scope (single host, VLAN, site, application).
2. **Establish** a theory (L1/L2/L3 path).
3. **Test** the theory (`ping`, `traceroute`, ARP table, MAC table, interface counters).
4. **Escalate/remediate** (config change, cable, ACL, routing).
5. **Document**.

**Tools mindset**: divide-and-conquer — confirm **physical**, then **link**, then **IP**, then **routing**, then **ACL/firewall**, then **application**.

---

## Domain 2 — Network Access (~20%)

### 2.1 Switching Concepts

- **MAC learning & flooding**: unknown unicast flooded; known unicast forwarded.  
- **MAC aging**, **static/dynamic** MAC entries.  
- **Collision/broadcast domain** placement.

### 2.2 VLANs & Trunking

- **802.1Q tagging**: native VLAN (untagged on trunk); **tagged** frames carry VID.  
- **Access ports**: single VLAN membership for user traffic.  
- **Voice VLAN** (concept): separate tagged voice traffic on access port.  
- **Inter-VLAN routing**: **SVI** (Switched Virtual Interface) on L3 switch or **router-on-a-stick** (subinterfaces).

### 2.3 VLAN Trunking Protocol (VTP) — Awareness

- Modes: server/client/transparent/off (implementation-dependent); **VTP can wipe VLANs if revision numbers surprise you** — many designs use **transparent/off** and manual VLAN provisioning.

### 2.4 Spanning Tree Protocol (STP) — Deep Enough for CCNA+Ops

**Problem**: loops in redundant L2 topology → broadcast storms.

**802.1D classic concepts** extended by **RSTP (802.1w)**:

- **Root bridge** election (lowest **bridge ID** = priority + MAC).  
- **Root ports**, **designated ports**, **blocking/alternate**.  
- **Port states** (learning/listening/forwarding/blocking — know RSTP equivalents).  
- **STP timers** (hello, forward delay, max age) — conceptual.

**Variations**:

- **PVST+** (Per-VLAN STP on Cisco).  
- **Rapid PVST+**.

**Best practices**: root placement near distribution/core; **spanning-tree portfast** on access ports; **BPDU guard**, **root guard**, **loop guard** (know purpose).

### 2.5 EtherChannel / Link Aggregation

- **LACP (802.3ad)** preferred vs **PAgP** (Cisco).  
- Load-balancing hash (src/dst MAC/IP/port) — affects elephant flows.  
- **Layer 2 vs Layer 3** port-channels (concept).

### 2.6 Wireless LAN Fundamentals (CCNA Scope)

- **SSID**, **BSSID**, **WLAN controllers** vs autonomous APs (architecture awareness).  
- **802.11** bands (2.4 vs 5 GHz), channels, **non-overlapping** channel planning at 2.4 GHz.  
- **Security**: WPA2/WPA3; **802.1X/EAP** conceptual flow for enterprise WLAN.

---

## Domain 3 — IP Connectivity (~25%)

### 3.1 Routing Concepts

- **Control plane vs data plane vs management plane** — exams love this framing.  
- **Forwarding decisions**: longest prefix match, **RIB → FIB** conceptually on Cisco platforms.

### 3.2 Static Routing

- **ip route** / IPv6 static equivalents; **floating static** with higher AD; **null routes** for blackholing aggregates.

### 3.3 OSPFv2 (Single-Area Focus for CCNA)

**Why OSPF**: link-state, scalable, standards-based IGP.

**Key mechanics:**

- **Hello** packets on `224.0.0.5` / `224.0.0.6`; **dead interval**.  
- **Neighbor adjacency** prerequisites: subnet, area ID, timers (often), stub flags, authentication if configured, **DR/BDR** on broadcast/non-broadcast segments.

**LSA concepts (high level):**

- Router LSA (Type 1), Network LSA (Type 2) in single-area thinking.

**Metrics**: **cost** derived from bandwidth (`reference_bandwidth` nuance).

**Configuration literacy**: `network` statements or interface-level activation (syntax varies by IOS-XE version — lab both patterns from your materials).

### 3.4 OSPFv3 / IPv6 Routing — Awareness

- Uses **link-local addresses** heavily; instances per IPv6 AF.

### 3.5 First Hop Redundancy — Conceptual

- **HSRP**, **VRRP**, **GLBP** — recognize roles (active/standby etc.), virtual MAC/IP, **preemption**.

---

## Domain 4 — IP Services (~10%)

### 4.1 NAT / PAT

- **Inside local/global**, **outside local/global** terminology.  
- **Static NAT**, **dynamic NAT**, **PAT (overload)** — port exhaustion awareness.  
- **NAT order** with ACLs (inside-to-outside translation points).

### 4.2 DHCP & DNS

- **DORA** (Discover, Offer, Request, Acknowledge).  
- **DHCP relay / ip helper-address**.  
- **DNS** A/AAAA records; split DNS concept.

### 4.3 SNTP/NTP

- **Stratum**, authentication (concept); accurate time critical for logs and certificates.

### 4.4 SNMP & Syslog

- **SNMPv2c vs v3** (community strings vs auth/priv).  
- **Trap vs inform**, **MIB** concept.  
- **Syslog** severity levels (0–7 mnemonic).

### 4.5 QoS — Foundational

- **Classification/marking** (DSCP, CoS).  
- **Congestion management**: queues; **policing vs shaping** distinction.  
- **Trust boundary** concept at access layer.

---

## Domain 5 — Security Fundamentals (~15%)

### 5.1 Security Concepts

- **CIA triad** (Confidentiality, Integrity, Availability).  
- **Threat vs vulnerability vs risk**.  
- **Defense in depth**, **least privilege**, **zero trust** (modern framing).

### 5.2 Device Hardening

- Strong passwords or **AAA**; **SSH-only** management; disable unused services; **privilege levels**; **CPU punting** awareness with heavy ACL/logging on small platforms.

### 5.3 Layer 2 Security

- **Port security**: sticky MAC, violation modes (protect/restrict/shutdown).  
- **DHCP snooping**, **DAI**, **IP source guard** — relationship: trusted vs untrusted ports.

### 5.4 ACLs

- **Standard vs extended**, numbered vs named.  
- **Implicit deny**.  
- **Inbound vs outbound** application direction relative to interface.  
- **IPv6 ACL** syntax differences (`ipv6 traffic-filter`).

### 5.5 Wireless Security

- WPA2-Enterprise vs WPA3 improvements (conceptual).

---

## Domain 6 — Automation & Programmability (~10%)

### 6.1 Controller-Based Networking

- **SDN** separation: control plane centralized vs distributed hybrid realities.  
- **Cisco DNA Center** role at high level (design, policy, assurance).

### 6.2 Data Encoding

- **JSON** structure for REST APIs; **YAML** for Ansible/playbooks.

### 6.3 REST APIs

- **HTTP verbs** GET/POST/PUT/PATCH/DELETE; status codes (200, 201, 400, 401, 404, 500).  
- **Authentication**: tokens, basic auth — lab-safe practice only.

### 6.4 Configuration Management Concepts

- **Idempotency**, **desired state**, inventory/host vars — Ansible conceptual fit.

### 6.5 Interpret Basic Python Automation

- Read scripts using `**requests`** or CLI wrappers; no need to be a software engineer, but **read/trace** code.

---

## Cross-Cutting Labs (Minimum Competency Checklist)

Complete analogous exercises until comfortable without notes:

1. **IPv4 VLSM design** on paper, then configure addressing on routers/L3 switch SVIs.
2. **VLAN + trunk + SVI** inter-VLAN routing; verify with `ping` across subnets.
3. **RSTP** topology change: unplug links, observe convergence behavior (log/mac moves).
4. **OSPF single-area** adjacency debug: break hello/dead, fix subnet mismatch, see stuck states.
5. **NAT overload** for internal clients to internet-facing interface; verify translations (`show ip nat translations`).
6. **ACL** blocking Telnet but permitting SSH; verify direction mistakes intentionally once.
7. **Port security** violation shutdown recovery (`err-disable recovery`).
8. **REST GET** against a lab-safe appliance returning JSON (even a mock server).

---

## Verification Command Literacy (IOS-XE Oriented)

Study families (not exhaustive):

- **Status**: `show ip interface brief`, `show ipv6 interface brief`  
- **L2**: `show vlan brief`, `show interfaces trunk`, `show spanning-tree`, `show etherchannel summary`, `show mac address-table`  
- **L3**: `show ip route`, `show ip protocols`, `show ip ospf neighbor`  
- **Services**: `show ip dhcp binding`, `show ip nat translations`, `debug` sparingly in lab  
- **Security**: `show access-lists`, `show port-security`

Understand **when** to use `**ping`/`traceroute`/`telnet`/`ssh`** for troubleshooting vs **show/debug**.

---

## Mental Models for “Advanced” Understanding

1. **Separate planes**: management (SSH/SNMP), control (routing protocols), data (forwarded frames/packets).
2. **Stateful vs stateless**: ACLs vs firewall sessions; NAT translations.
3. **Symmetric vs asymmetric routing** pitfalls with NAT and firewalls.
4. **MTU & fragmentation**: IPv4 DF bit and PMTUD concepts; avoid black holes.
5. **Bufferbloat** vs **microburst** drops — conceptual QoS motivation.

---

## Official & High-Quality Supplementary Resources

- **Cisco Learning**: CCNA learning path and exam topics page for **200-301** (always verify current blueprint).  
- **Cisco Networking Academy** — structured cohort curriculum.  
- **RFCs** (selected reading): 791 (IPv4), 8200 (IPv6), 793 (TCP), 2328/5340 (OSPF v2/v3 concepts — skim at CCNA depth).  
- **Books**: Cisco Press **Official Cert Guide** for **200-301** (volume alignment varies by edition).

---

## Appendix A — Quick IPv4 Subnet Reference Patterns


| Prefix | Mask            | Usable hosts per subnet |
| ------ | --------------- | ----------------------- |
| /24    | 255.255.255.0   | 254                     |
| /25    | 255.255.255.128 | 126                     |
| /26    | 255.255.255.192 | 62                      |
| /27    | 255.255.255.224 | 30                      |
| /28    | 255.255.255.240 | 14                      |
| /29    | 255.255.255.248 | 6                       |
| /30    | 255.255.255.252 | 2 (typical P2P)         |


---

## Appendix B — OSI ↔ Cisco Troubleshooting Cheat


| Symptom cluster                      | Likely layer                              |
| ------------------------------------ | ----------------------------------------- |
| Link light down, errors climbing     | L1/L2                                     |
| ARP fails, wrong VLAN                | L2                                        |
| Ping gateway fails from host         | L2/L3 on segment                          |
| Ping gateway OK, remote subnet fails | L3 routing                                |
| Routing OK, app fails                | Transport (ports/firewall) or application |


---

## Appendix C — Exam-Day Discipline

- **Flag** simlet items and manage time; simulations can consume disproportionate minutes.  
- **Read the question twice** for “choose two” / negation (“which is **not**”).  
- **Picture the topology** implied by the question before selecting answers.

---

**Disclaimer:** Exam codes, percentages, and precise topics change. Before registering, confirm the latest **200-301** exam topics on Cisco’s official certification pages and adjust your study accordingly.

**Good luck.** Competence in networking comes from **consistent lab proof** — make every topic earn its place by working on real (or simulated) configurations.