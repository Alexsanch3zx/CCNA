# Networking Basics for Cybersecurity

**Purpose:** Build enough networking literacy to understand **how attacks traverse networks**, how **defenders segment and monitor**, and how **logs and packets** relate to real incidents—without assuming prior depth beyond general curiosity.

**Ethics:** Use these concepts only on systems and networks you **own** or have **explicit written authorization** to test.

---

## 1. Why Networking Matters in Security

Every compromise has a **path**: hosts talk using **addresses**, **ports**, and **protocols** across **links** and **zones**. Security roles rely on this map to:

- **Contain** blast radius (segmentation, firewalls).  
- **Detect** abuse (traffic patterns, IDS/IPS, flow logs).  
- **Respond** (isolate VLANs, revoke VPN, block IOCs at perimeter).

If you cannot picture **where a packet could go**, you cannot judge whether a control actually helps.

---

## 2. Layered Models (Defense-Relevant View)

### 2.1 OSI — Seven Layers (Reference)


| Layer | Name         | Security hooks (examples)                               |
| ----- | ------------ | ------------------------------------------------------- |
| 7     | Application  | HTTP abuse, SMTP phishing relays, malformed DNS queries |
| 6     | Presentation | Encoding/decryption boundaries (often folded into apps) |
| 5     | Session      | Session fixation, teardown abuse                        |
| 4     | Transport    | Port scans, SYN floods, UDP amplification               |
| 3     | Network      | IP spoofing, routing manipulation, ICMP tunnels         |
| 2     | Data Link    | ARP spoofing, MAC flooding, VLAN hopping concepts       |
| 1     | Physical     | Tap, rogue AP, evil maid hardware                       |


You will hear **“Layer 7 firewall”** or **“Layer 3 segmentation”**—those phrases anchor on this stack.

### 2.2 TCP/IP Model (How Practitioners Speak)


| Layer       | Maps to OSI | Typical protocols           |
| ----------- | ----------- | --------------------------- |
| Application | 5–7         | HTTP/S, DNS, SMTP, SSH, SMB |
| Transport   | 4           | TCP, UDP                    |
| Internet    | 3           | IPv4, IPv6, ICMP            |
| Link        | 1–2         | Ethernet, Wi‑Fi             |


**Mnemonic for incident triage:** start **high** (what application broke?) and validate **down** (DNS? routing? cable?), or start **low** when outages are total.

---

## 3. Addresses, Zones, and Attack Surface

### 3.1 IPv4 Addresses

- **32-bit** address, written as four decimals (e.g. `203.0.113.10`).  
- **Public** addresses are globally routable on the Internet; **private** ranges stay inside orgs (**RFC 1918**):  
  - `10.0.0.0/8`  
  - `172.16.0.0/12`  
  - `192.168.0.0/16`
- **NAT** maps many internal hosts to fewer public IPs—changes **where** logging sees traffic and **how** inbound attacks target services (**DMZ**, port forwarding).

**Security angle:** lateral movement often stays **private IP to private IP**; perimeter alerts miss east‑west paths unless internal telemetry exists.

### 3.2 CIDR Notation (`/24`, `/16`, …)

`/24` means **first 24 bits** identify the network; remaining bits identify hosts inside that subnet.


| Prefix | Hosts (approx usable IPv4) | Typical use                 |
| ------ | -------------------------- | --------------------------- |
| /32    | 1 host                     | Single endpoint identifier  |
| /30    | 2 usable                   | Point-to-point links        |
| /24    | ~254                       | Common VLAN / office subnet |
| /16    | ~65k                       | Large campus aggregate      |


**Security angle:** firewall rules and detection scopes are written in **CIDR**. Mis-sizing subnets affects **broadcast noise** and **ACL fatigue**.

### 3.3 IPv6 (Essentials)

- **128-bit** addresses; vastly larger space reduces casual scanning effectiveness—**not** “IPv6 is secure.”  
- **Link-local** (`fe80::/10`) stays on-link; global addresses route like IPv4 conceptually.  
- Transition mechanisms (**dual stack**, tunnels) add **edges** defenders must inventory.

---

## 4. Ports, Sockets, and Services

### 4.1 TCP vs UDP


|                | TCP                                        | UDP                                     |
| -------------- | ------------------------------------------ | --------------------------------------- |
| Connection     | Connection-oriented (handshake)            | Connectionless                          |
| Reliability    | Retransmits, ordering                      | Best effort                             |
| Abuse patterns | Half-open scans, session hijack discussion | Amplification floods, spoofable sources |


A **socket** is `**IP:port`** paired with remote `**IP:port`** for a conversation context.

### 4.2 Well-Known Ports (Know These Cold)


| Port(s)   | Protocol | Security notes                                                                 |
| --------- | -------- | ------------------------------------------------------------------------------ |
| 20/21     | FTP      | Cleartext credentials unless FTPS; bounce abuse legacy                         |
| 22        | SSH      | Brute force, key theft; jump hosts targeted                                    |
| 23        | Telnet   | Cleartext—flag as legacy risk                                                  |
| 25        | SMTP     | Spam relays, phishing pipelines                                                |
| 53        | DNS      | Critical dependency; poisoning/cache abuse; DNS tunneling                      |
| 67/68     | DHCP     | Rogue DHCP MITM scenarios                                                      |
| 80        | HTTP     | Cleartext; redirects often lead to HTTPS                                       |
| 443       | HTTPS    | TLS inspection policy debates; still malware C2 channel                        |
| 445       | SMB      | Lateral movement (**WannaCry-class** worm paths); never expose raw to Internet |
| 3389      | RDP      | Very commonly brute forced or exploited                                        |
| 5985/5986 | WinRM    | Post‑ex lateral movement                                                       |


Ephemeral ports are **high-numbered client-side** ports chosen per outbound session—visible in firewall logs alongside destination `:443`.

---

## 5. Ethernet and Switching (What Happens on the LAN)

### 5.1 Frames and MAC Addresses

Switches forward **frames** using **destination MAC addresses**. They learn **which MAC lives off which port** by observing traffic.

### 5.2 ARP (Address Resolution Protocol)

IPv4 resolves `**next-hop IP → MAC`** on the local segment via ARP.

**ARP spoofing / poisoning:** attacker answers ARP queries falsely—traffic flows **through attacker’s machine** → classic **MITM** on switched LANs **without** proper controls.

**Mitigations (conceptual):** dynamic ARP inspection (**DAI**) with **DHCP snooping**, port security, **802.1X**.

### 5.3 VLANs

**Virtual LANs** logically segment switches at Layer 2. **Misconfiguration** (native VLAN mismatch, overly broad VLAN membership) expands lateral movement options.

---

## 6. Routing (Getting Between Subnets)

**Routers** forward **packets** between IP subnets using **routing tables**: longest-prefix match wins.

**Security relevance:**

- **Default route (`0.0.0.0/0`)** sends traffic to Internet egress—monitor what **may** leave.  
- **Asymmetric routing** can break **stateful firewalls** or cause **silent drops**.  
- **BGP hijacking** is Internet-scale routing manipulation—mostly relevant at ISP/cloud architect level but informs **supply chain trust**.

---

## 7. DNS — The Protocol Everyone Depends On

### 7.1 What DNS Does

Maps **names → records** (A/AAAA for addresses, MX for mail, TXT for many things including security proofs).

### 7.2 DNS Abuse Patterns

- **Cache poisoning / spoofing** — false answers cached downstream.  
- **Tunneling** — encapsulating data inside DNS queries/responses (often **slow**, sometimes **covert**).  
- **Fast flux / DGAs** — rapidly changing resolutions for resilient malware infrastructure.  
- **Typosquatting / homograph domains** — human deception layered on DNS.

**Defense artifacts:** resolver logs, **DNS firewall** policies, response policy zones (**RPZ**), encrypted DNS (**DoH/DoT**) privacy vs visibility tradeoffs.

---

## 8. HTTP and HTTPS (Application Layer Entry)

### 8.1 HTTP

Cleartext request/response—easy to intercept on shared networks **without encryption**.

### 8.2 TLS (often called SSL)

Provides **confidentiality** and **integrity** on top of TCP for HTTPS—and many other protocols (**SMTPS**, **LDAPS**, etc.).

**Security operations angles:**

- **Certificates** tie identity claims to public keys; **mis-issued** or **stolen** certs break trust.  
- **TLS inspection** at proxies decrypts traffic for scanning—powerful **visibility**, heavy **privacy/policy** implications.  
- **Cipher suites** deprecated over time (**weak crypto** findings).

---

## 9. Firewalls and Network Controls

### 9.1 Stateless vs Stateful

- **Stateless:** matches rule per packet—fast, dumb context.  
- **Stateful:** tracks **connections** (TCP states, pseudo-flow for UDP)—common **NGFW** baseline.

### 9.2 ACLs vs NGFW / WAF


| Control  | Typical layer            | Focus                             |
| -------- | ------------------------ | --------------------------------- |
| ACL      | L3/L4                    | Permit/deny IP/port               |
| Firewall | L3/L4 (+ App-ID in NGFW) | Policy enforcement, zones         |
| IDS/IPS  | Often L3–L7 visibility   | Detect/block signatures/anomalies |
| WAF      | HTTP/S application edge  | OWASP-class web abuse             |


**Zones:** **inside**, **outside**, **DMZ**—policy expresses **trust boundaries**.

---

## 10. VPN and Tunneling

**VPNs** encapsulate traffic—remote users appear **inside** corporate routing (policy permitting).

**Security framing:**

- VPN credential theft → broad access.  
- Split tunnel vs full tunnel affects **what** egress monitoring sees on endpoints.  
- **Site‑to‑site VPN** connects networks—misconfigured crypto domains leak paths.

Other tunnels (**GRE**, **IPsec**, **WireGuard**) appear in hybrid cloud designs—know enough to **read diagrams**.

---

## 11. Wireless Basics for Security

- **SSID** is human label; **802.11** frames still bridge into wired LAN eventually.  
- **Open Wi‑Fi** = trivial passive observation unless apps use TLS everywhere.  
- **Evil twin AP:** impersonates legitimate SSID—credential phishing **via captive portals** still hooks victims.  
- **WPA2‑Enterprise** uses **802.1X**—better identity binding than shared PSK homes.

---

## 12. Network Evidence You Will Actually See

### 12.1 Flow Logs / NetFlow / IPFIX

Summaries: **who talked to whom**, ports, bytes, duration—cheap at scale for hunting **beacons**, **scans**, **exfil spikes**.

### 12.2 PCAP (Packet Capture)

Full packets—heavy storage; gold standard for **payload proof** when lawful and feasible (**Wireshark**, **tcpdump**).

### 12.3 Firewall / Proxy Logs

Often first source for **blocked C2**, **Tor exits**, **geo anomalies**.

---

## 13. Attack Patterns Mapped to Networking


| Technique category | Networking layer      | Brief mechanism                       |
| ------------------ | --------------------- | ------------------------------------- |
| Port scan          | Transport             | Discover listening services           |
| SYN flood          | Transport             | Exhaust connection tables             |
| UDP amplification  | Transport/IP          | Spoofed source triggers large replies |
| MITM on LAN        | Data Link             | ARP poisoning or rogue DHCP           |
| DNS poisoning      | Application/DNS infra | Bad resolver answers                  |
| BGP hijack         | Routing               | Paths drawn to attacker AS            |
| Exfiltration       | Any allowed egress    | Blend into normal HTTPS/DNS           |


Understanding **layer** clarifies **which control class** applies first.

---

## 14. Segmentation & Zero Trust (Modern Baseline Language)

**Segmentation:** VLANs, firewalls between tiers (**web / app / DB**), **microsegmentation** with host agents.

**Zero Trust (network slice):** assume breach—**authenticate** sessions, **authorize** least privilege continuously, **inspect** logs—**not** “trust intranet IP forever.”

---

## 15. Study Path — Turn Basics Into Skill

1. **Draw** your home lab network: IPs, gateway, DNS resolver path.
2. **Trace** one HTTPS browse to `curl -v` resolution → TCP handshake → TLS (`openssl s_client`) in lab only.
3. **Read** a **tcpdump** or **Wireshark** capture of DNS + TLS setup—identify five‑tuple (**proto, src IP, src port, dst IP, dst port**).
4. **Write** three firewall rules in plain English then translate to vendor syntax when you lab.
5. **Explain** ARP spoofing to a peer **without** jargon wall—then list two mitigations.

---

## Appendix — Hexadecimal / Binary Tip (For Reading Addresses and Flags)

IPv4 octets map from **8-bit binary** to decimal (0–255). TLS cipher suites, TCP flags in captures, and IPv6 shorthand all assume comfort flipping **binary ↔ hex**—practice eight bits at a time.

---

## Appendix — Minimal Tool Literacy (Authorized Use Only)


| Tool                     | Role                                |
| ------------------------ | ----------------------------------- |
| **ping**                 | Reachability & ICMP handling policy |
| **traceroute / tracert** | Path asymmetry hints                |
| **nslookup / dig**       | DNS answers vs resolver trust       |
| **netstat / ss**         | Local listeners and sockets         |
| **curl**                 | HTTP/S semantics replay             |
| **nmap**                 | Discovery (**permission required**) |
| **Wireshark**            | Protocol decode discipline          |


---

**Bottom line:** cybersecurity networking basics are **not** trivia—they are the coordinate system for **visibility**, **control**, and **story** during incidents. Depth grows toward **traffic analysis**, **cloud networking**, and **detection engineering** from here.