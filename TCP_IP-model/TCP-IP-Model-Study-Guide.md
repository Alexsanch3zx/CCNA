# TCP/IP Model — CCNA Study Guide

**Audience:** CCNA (exam **200-301**) candidates who need to **understand**, not just memorize, how data moves across a network.

**How to use this guide:** Read Parts 1–4 in order, then test yourself with Part 8. For each layer, you should answer: *What does it do? What device cares? What PDU is used? What breaks here?*

---

## Part 1 — Why the TCP/IP Model Matters for CCNA

### 1.1 Two models, one job

Networks are taught with two frameworks:

| Model | Layers | Role in your studies |
| ----- | ------ | -------------------- |
| **OSI** | 7 | Fine-grained **reference** (exam vocabulary: "Layer 3") |
| **TCP/IP** | 4 | How **real stacks** on hosts and routers are organized |

**CCNA tests both.** Cisco IOS, Windows, Linux, and the Internet run on **TCP/IP**. When someone says "Layer 3 firewall," they mean OSI; when someone says "the Internet layer," they mean TCP/IP.

### 1.2 The four TCP/IP layers (memorize this stack)

```text
+----------------------------------+
|  4. Application                  |  HTTP, DNS, SSH, DHCP (messages)
+----------------------------------+
|  3. Transport                    |  TCP, UDP (segments / datagrams)
+----------------------------------+
|  2. Internet                     |  IPv4, IPv6, ICMP (packets)
+----------------------------------+
|  1. Network Access (Link)        |  Ethernet, Wi-Fi (frames)
+----------------------------------+
```

**Bottom = closest to the wire. Top = closest to the user/application.**

### 1.3 One sentence per layer (exam-ready)

| Layer | One-line purpose |
| ----- | ---------------- |
| **Network Access** | Put bits on the medium; deliver **frame to frame** on one link (MAC). |
| **Internet** | Address endpoints logically; **route packets** across many hops. |
| **Transport** | Deliver data **host-to-host** using **ports**; optional reliability. |
| **Application** | Meaningful services users and apps use (web, mail, name resolution). |

---

## Part 2 — TCP/IP Mapped to OSI (Do Not Mix These Up)

| TCP/IP layer | Maps to OSI | Typical PDU name (CCNA) |
| ------------ | ----------- | ------------------------ |
| Application | Layers 5, 6, 7 | Data |
| Transport | Layer 4 | **Segment** (TCP) or **Datagram** (UDP) |
| Internet | Layer 3 | **Packet** |
| Network Access | Layers 1 and 2 | **Frame** (L2) / **Bits** (L1) |

**Memory trick:** As you go **down** the stack, each layer **wraps** the layer above. As you go **up**, each layer **unwraps**.

---

## Part 3 — Each Layer in Detail

---

### Layer 1 (bottom): Network Access — Physical + Data Link

**Also called:** Link layer, network interface layer.

#### What problem it solves

Gets a chunk of bits from **one interface to the next** on the **same** broadcast domain or point-to-point link. It does **not** route across the whole Internet by itself.

#### What lives here (CCNA focus)

| Topic | What to know |
| ----- | ------------ |
| **Ethernet** | MAC addresses, switches forward by MAC |
| **802.1Q** | VLAN tags on trunks |
| **ARP** (IPv4) | Resolves **next-hop IP** to **MAC** on the local segment |
| **Cabling / duplex** | Link up/down, speed mismatch |
| **Wi-Fi** | Still ends up bridged into wired Ethernet eventually |

#### Devices that operate mainly here

- **Switch** (Layer 2)
- **Wireless AP** (radio + bridge to Ethernet)
- **NIC** on PC/server

#### PDU: Frame

A **frame** includes:

- Source MAC and destination MAC
- Type/length (often tells you IPv4 vs IPv6)
- Payload (usually an IP **packet**)
- FCS (error check at end)

#### CCNA troubleshooting at this layer

| Symptom | Think link layer |
| ------- | ---------------- |
| Link light down | Cable, port shutdown, wrong media |
| Same VLAN cannot ping | Wrong VLAN, trunk missing, MAC not learned |
| Works on L2 but not beyond subnet | Not a pure L2 problem — check gateway (L3) |

**Verify commands (switch):**

```cisco
show interfaces status
show mac address-table
show vlan brief
show interfaces trunk
```

---

### Layer 2: Internet Layer (Do not confuse with "Internet" the web)

In TCP/IP naming, **Internet layer = OSI Layer 3**.

#### What problem it solves

Delivers **packets** between hosts using **logical addresses** (IPv4/IPv6), possibly across **many routers**.

#### What lives here

| Protocol / concept | Role |
| ------------------ | ---- |
| **IPv4 / IPv6** | Addressing, TTL/Hop Limit, fragmentation rules |
| **ICMP** | Diagnostics (`ping`, `traceroute`), errors |
| **Routing** | OSPF, static routes — **control plane** builds the table; **forwarding** uses the table |
| **ARP** | Often described with L2; **used** when sending IP to a neighbor on-link |

#### Devices that operate here

- **Router** (primary)
- **Layer 3 switch** (routes between VLANs with SVIs)

#### PDU: Packet

An IP **packet** includes:

- Source IP, destination IP
- Protocol field (tells transport: TCP = 6, UDP = 17, ICMP = 1)
- Payload (TCP segment, UDP datagram, or ICMP message)

#### Key CCNA ideas

1. **Same subnet** — host sends to destination MAC after ARP (or uses cached ARP).
2. **Different subnet** — host sends packet to **default gateway** MAC; router forwards.
3. **Longest prefix match** — router picks the most specific route in the table.

**Verify commands (router):**

```cisco
show ip interface brief
show ip route
show ip arp
ping <ip>
traceroute <ip>
```

---

### Layer 3: Transport Layer

#### What problem it solves

Multiple applications on one host share one IP address. Transport uses **ports** to identify **which application** sends or receives data.

#### TCP vs UDP (must know cold for CCNA)

| | **TCP** | **UDP** |
| - | ------- | ------- |
| Connection | Connection-oriented (3-way handshake) | Connectionless |
| Reliability | Acknowledgments, retransmit | Best effort |
| Ordering | Yes | No guarantee |
| Overhead | Higher | Lower |
| Examples | HTTP/HTTPS, SSH, SMTP, FTP control | DNS (often), DHCP, VoIP, QUIC (over UDP) |
| PDU name | **Segment** | **Datagram** |

#### TCP three-way handshake (exam favorite)

```text
Client ---- SYN ------------------------> Server
Client <--- SYN-ACK -------------------- Server
Client ---- ACK ------------------------> Server
```

- **SYN** — synchronize sequence numbers  
- **ACK** — acknowledge received data  
- Connection is **established** after the third step

#### TCP teardown (simplified)

FIN and ACK exchanges close the session gracefully. Know that **half-open** or **RST** can appear when apps crash or ports are closed.

#### Port numbers

| Range | Name | Example |
| ----- | ---- | ------- |
| 0–1023 | Well-known | 80 HTTP, 443 HTTPS, 22 SSH |
| 1024–49151 | Registered | Many vendor apps |
| 49152–65535 | Ephemeral | Client-side source ports in outbound sessions |

**Socket** = `(source IP, source port, dest IP, dest port, protocol)` — identifies a conversation.

#### CCNA troubleshooting at transport layer

| Symptom | Think transport |
| ------- | --------------- |
| `ping` works but web fails | Port 80/443 blocked, app down, wrong URL |
| Connection timeout | Firewall ACL, no listener on port, routing OK but TCP SYN dropped |
| Works with `telnet host 22` but not 23 | Service-specific (SSH vs Telnet) |

**Host check (conceptual):** `netstat` / `ss` shows listening ports; `telnet` or `nc` tests TCP reachability to a port.

---

### Layer 4 (top): Application Layer

#### What problem it solves

Provides **network services** that applications use: name lookup, file transfer, remote login, web pages.

#### What lives here (representative protocols)

| Protocol | Port(s) | What it does |
| -------- | ------- | ------------ |
| **HTTP/HTTPS** | 80 / 443 | Web |
| **DNS** | 53 UDP/TCP | Name to IP |
| **DHCP** | 67/68 | Automatic IPv4 addressing |
| **SSH** | 22 | Secure remote shell |
| **SMTP** | 25 | Email transfer |
| **SNMP** | 161 | Network management |
| **FTP** | 20/21 | File transfer (legacy) |

**Note:** TLS/SSL for HTTPS often sits **inside** the application stack as a library — CCNA may still call it "Application layer" in the TCP/IP model.

#### CCNA troubleshooting at application layer

| Symptom | Think application |
| ------- | ----------------- |
| IP works, name fails | DNS wrong or down |
| Browser error but ping OK | HTTP/S service, certificate, proxy |
| No address on PC | DHCP scope, relay, or pool exhausted |

---

## Part 4 — Encapsulation: Follow One Ping

**Scenario:** PC0 `192.168.10.10` pings PC1 `192.168.10.11` on the same VLAN.

### Step-by-step (sending direction)

```text
Application:  "I need to send ICMP echo request"
Transport:    (ICMP is inside IP; no TCP/UDP for ping)
Internet:     Build IP packet: src 10.10, dst 10.11, proto ICMP
Network Access: ARP for 10.11 if needed -> build Ethernet frame
                dst MAC = PC1, src MAC = PC0
Physical:     Encode as bits on the wire
```

### At the switch

- Reads **destination MAC** only (L2)
- Forwards frame out the correct port
- Does **not** change IP addresses

### At the receiving PC

```text
Physical:       Receive bits
Network Access: Check FCS, check MAC is for me, strip Ethernet header
Internet:       Check IP is for me, pass ICMP to stack
Application/ICMP: Reply to ping
```

**Rule:** Each hop **de-encapsulates** up to the layer it understands, then **re-encapsulates** down when forwarding (routers re-encapsulate at L2 for the **next hop**).

### Cross-subnet ping (adds a router)

1. PC sends packet to **default gateway** IP.  
2. Frame destination MAC = **router's** MAC on that VLAN.  
3. Router strips L2, routes by **destination IP**, picks outgoing interface.  
4. Router ARPs (or uses cache) for **next-hop** MAC, builds **new** frame.  
5. Destination PC receives frame.

**CCNA skill:** Draw this on paper before the exam.

---

## Part 5 — Where Cisco Features Sit in TCP/IP

| Feature | Primary TCP/IP layer | Notes |
| ------- | -------------------- | ----- |
| VLAN / trunk | Network Access | Logical L2 segmentation |
| STP | Network Access | Prevents L2 loops |
| SVI / routed port | Internet | L3 gateway on switch |
| Static route / OSPF | Internet (control plane) | OSPF messages are IP packets |
| NAT | Internet (and policy) | Alters IP/port headers |
| ACL (extended) | Often Internet + Transport | Matches IP and ports |
| Port security | Network Access | MAC limits on switch port |
| DHCP / DNS | Application | Configured on router or server |

---

## Part 6 — Troubleshooting Top-Down vs Bottom-Up

### Bottom-up (CCNA lab style)

1. **Link up?** (`show interfaces`, link lights)  
2. **Correct VLAN / trunk?**  
3. **IP address and mask correct?**  
4. **Default gateway reachable?**  
5. **Routing table has path?** (`show ip route`)  
6. **ACL or firewall blocking?**  
7. **Application / DNS / DHCP?**

### Top-down (user complaint: "website broken")

1. Browser / URL / DNS  
2. TCP 443 reachable?  
3. Routing and NAT  
4. L2 path

**Both are valid.** Use the symptom to pick a starting layer.

---

## Part 7 — Common CCNA Exam Traps

| Trap | Truth |
| ---- | ----- |
| "TCP/IP has 7 layers" | **OSI** has 7; **TCP/IP** has **4** |
| "Routers forward by MAC" | Routers forward by **IP**; rewrite **MAC** each hop |
| "Switches route IP" | Default switch is **L2** unless L3 / SVI configured |
| "UDP is always faster so always better" | UDP has **no reliability**; apps must handle loss |
| "Ping uses TCP" | **ICMP** over IP (no TCP) |
| "Port 80 and 443 are transport protocols" | They are **port numbers**; HTTP/HTTPS are application protocols over TCP |

---

## Part 8 — Self-Check (Answer Without Looking)

### Questions

1. List the four TCP/IP layers from bottom to top.  
2. What PDU does TCP use at the transport layer?  
3. Which layer adds source and destination MAC addresses?  
4. Why does a host ARP its default gateway when pinging a remote server?  
5. Map **DNS** to TCP/IP layer and default port.  
6. Name three protocols at the **Internet** layer.  
7. What happens to IP addresses when a packet crosses a router?  
8. What happens to Ethernet MAC addresses when a packet crosses a router?

### Answers

1. Network Access, Internet, Transport, Application.  
2. **Segment**.  
3. **Network Access** (data link portion).  
4. Remote IP is off-subnet; L2 delivery requires the **gateway's MAC**.  
5. **Application** layer; port **53** (UDP common, TCP for large responses/zone transfers).  
6. IPv4, IPv6, ICMP (also accept routing protocols carried in IP for discussion).  
7. **Unchanged** (unless NAT modifies them).  
8. **Changed** — new frame built with next-hop source/dest MAC.

---

## Part 9 — Quick Comparison Table (Print This)

| Layer | PDU | Addressing | Device | CCNA verify |
| ----- | --- | ---------- | ------ | ----------- |
| Application | Data | Names, URLs | Host, server | DNS, DHCP, browser |
| Transport | Segment / Datagram | TCP/UDP ports | Host | `telnet host port`, socket |
| Internet | Packet | IPv4/IPv6 | Router, L3 switch | `ping`, `show ip route` |
| Network Access | Frame / Bits | MAC, VLAN | Switch, NIC | `show mac`, `show vlan` |

---

## Part 10 — Study Path (One Week)

| Day | Task |
| --- | ---- |
| 1 | Draw 4 layers; write PDU per layer from memory |
| 2 | Trace same-subnet ping on paper (MAC + IP) |
| 3 | Trace cross-subnet ping through one router |
| 4 | Compare TCP vs UDP; list 5 apps each |
| 5 | Packet Tracer: capture Simulation mode for DNS + HTTP |
| 6 | Break lab (wrong gateway); fix using bottom-up |
| 7 | Retake Part 8 self-check under 10 minutes |

---

## Related Files in This Repo

- [CCNA-Networking-Study-Guide.md](../CCNA-Networking-Study-Guide.md) — full CCNA syllabus  
- [networking-basics.md](../networking-basics.md) — security-oriented networking primer  
- [IP-addresses/subnetting.md](../IP-addresses/subnetting.md) — IPv4 subnetting (Internet layer math)  
- [cheat-sheets/command-cheat-sheet.md](../cheat-sheets/command-cheat-sheet.md) — IOS verification commands  
- [labs/](../labs/) — Packet Tracer labs  

---

**Bottom line for CCNA:** The TCP/IP model tells you **where** addressing, routing, ports, and frames each belong. When troubleshooting, name the layer first — then use the right **show** command or host test for that layer.
