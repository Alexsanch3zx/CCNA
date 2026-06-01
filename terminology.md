# Networking Terminology — CCNA & Study Guide Glossary

Quick definitions for terms used across this repo. Terms are grouped by topic, then listed **A–Z** within each group.

---

## How to Use This File

- **Studying:** Read one section per day and explain each term in your own words.  
- **Labs:** Look up a term when a guide or CLI output uses jargon you do not recognize.  
- **Exam prep:** Focus on sections matching weak blueprint areas (switching, routing, IP services, security).

---

## General & Models

| Term | Definition |
| ---- | ---------- |
| **ACK** | Acknowledgment; TCP flag confirming received data. |
| **ACL** | Access Control List; permit/deny rules for traffic, often on router interfaces or VLANs. |
| **API** | Application Programming Interface; how programs request services (relevant to network automation). |
| **ARP** | Address Resolution Protocol; maps IPv4 address to MAC address on the same Layer 2 segment. |
| **AS** | Autonomous System; collection of networks under one administrative policy (BGP context). |
| **Bandwidth** | Data rate capacity of a link, typically in bits per second (bps, Mbps, Gbps). |
| **Broadcast** | Traffic sent to all hosts on a Layer 2 domain; IPv4 limited broadcast is `255.255.255.255`. |
| **Broadcast domain** | Set of devices that receive each other's broadcast frames; segmented by routers and VLANs. |
| **CCNA** | Cisco Certified Network Associate; associate-level certification (exam **200-301**). |
| **CIDR** | Classless Inter-Domain Routing; notation like `/24` showing prefix length. |
| **Collision domain** | Segment where Ethernet collisions could occur (legacy hubs); one per switch port in full duplex. |
| **Encapsulation** | Process of adding headers (and sometimes trailers) as data moves down the protocol stack. |
| **Frame** | Layer 2 PDU; includes MAC addresses and payload (often an IP packet). |
| **Gateway** | Router interface IP used by hosts as the next hop to reach off-subnet destinations (default gateway). |
| **ISO/OSI model** | Seven-layer reference model (Physical through Application). |
| **Latency** | Delay for data to cross a network path. |
| **MTU** | Maximum Transmission Unit; largest IP packet size allowed on a link (often 1500 bytes on Ethernet). |
| **Multicast** | One-to-many delivery; IPv4 Class D; Ethernet multicast MAC range. |
| **Node** | Any networked device (host, router, switch, server). |
| **OSI model** | See **ISO/OSI model**. |
| **Packet** | Layer 3 PDU; IP datagram with source/destination IP addresses. |
| **PDU** | Protocol Data Unit; name for data at each layer (frame, packet, segment, etc.). |
| **Protocol** | Rules for communication between devices (TCP, OSPF, HTTP, etc.). |
| **Segment** | TCP transport-layer PDU. |
| **TCP/IP model** | Four-layer practical model: Network Access, Internet, Transport, Application. |
| **Throughput** | Actual useful data rate achieved (often less than bandwidth). |
| **Unicast** | One-to-one delivery to a single destination. |

---

## Cabling & Physical Layer

| Term | Definition |
| ---- | ---------- |
| **Copper (UTP)** | Twisted-pair cable for Ethernet; Cat5e/Cat6 common. |
| **Crossover cable** | Pair swap for like devices (historical); often replaced by auto-MDIX. |
| **DCE / DTE** | Data Circuit-terminating Equipment vs Data Terminal Equipment; serial WAN clocking on DCE. |
| **Duplex** | Full duplex (send/receive simultaneously) vs half duplex. |
| **Fiber** | Optical cable; single-mode (long distance) vs multimode (shorter, campus). |
| **MDIX** | Medium Dependent Interface Crossover; auto-crossing on modern ports. |
| **PoE** | Power over Ethernet; delivers power to APs, phones, cameras on data cable. |
| **Straight-through cable** | Standard Ethernet cable for host-to-switch or switch-to-router links. |

---

## Ethernet & Switching (Layer 2)

| Term | Definition |
| ---- | ---------- |
| **802.1Q** | VLAN tagging standard on trunk links. |
| **Access port** | Switch port assigned to a single VLAN; untagged user traffic. |
| **BPDU** | Bridge Protocol Data Unit; STP messages between switches. |
| **Bridge ID** | STP identifier (priority + MAC); lowest wins root election. |
| **CAM table** | Content Addressable Memory; switch MAC address table. |
| **DAI** | Dynamic ARP Inspection; validates ARP against trusted DHCP bindings. |
| **DHCP snooping** | Switch feature trusting DHCP server ports and blocking rogue servers. |
| **EtherChannel** | Bundling multiple links into one logical L2/L3 port-channel. |
| **FCS** | Frame Check Sequence; trailer for Ethernet error detection. |
| **LACP** | Link Aggregation Control Protocol; IEEE standard for EtherChannel (802.3ad). |
| **MAC address** | 48-bit hardware address; written in hex (e.g., `AA:BB:CC:DD:EE:FF`). |
| **Native VLAN** | Untagged VLAN on an 802.1Q trunk; must match on both ends. |
| **Port security** | Limits MAC addresses allowed on a switch access port. |
| **Root bridge** | STP reference switch with lowest bridge ID; center of spanning tree. |
| **RSTP** | Rapid Spanning Tree Protocol (802.1w); faster convergence than classic STP. |
| **Spanning Tree Protocol (STP)** | Prevents Layer 2 loops by blocking redundant paths. |
| **SVI** | Switched Virtual Interface; VLAN interface on a Layer 3 switch for routing. |
| **Trunk** | Link carrying multiple VLANs with 802.1Q tags. |
| **VLAN** | Virtual LAN; logical broadcast domain on a switch. |
| **VTP** | VLAN Trunking Protocol; Cisco protocol to sync VLAN info (often disabled in production). |

---

## IPv4 Addressing & Subnetting

| Term | Definition |
| ---- | ---------- |
| **Broadcast address** | Last address in a subnet; host bits all 1; not assigned to a host. |
| **Default route** | Route `0.0.0.0/0`; gateway of last resort to the Internet or core. |
| **Host bits** | Bits in an address identifying the host within a subnet. |
| **IPv4** | 32-bit Internet Protocol version 4 (e.g., `192.168.1.1`). |
| **Network address** | First address in subnet; host bits all 0. |
| **Network bits** | Bits identifying the subnet per the mask. |
| **Octet** | 8-bit group in dotted-decimal IPv4 (four octets total). |
| **Private address** | RFC 1918 space not routed on public Internet (`10/8`, `172.16/12`, `192.168/16`). |
| **Public address** | Globally routable IPv4 on the Internet. |
| **Subnet mask** | Defines network vs host bits (e.g., `255.255.255.0` = `/24`). |
| **Subnetting** | Dividing a network into smaller subnets with longer prefixes. |
| **Supernetting** | Summarizing contiguous subnets into one larger prefix (route aggregation). |
| **Usable hosts** | Host addresses excluding network and broadcast (typical LAN subnets). |
| **VLSM** | Variable Length Subnet Masking; different prefix lengths in one address plan. |
| **Wildcard mask** | Inverse of subnet mask bits; used in OSPF and ACL matching. |

---

## IPv6

| Term | Definition |
| ---- | ---------- |
| **Dual stack** | Running IPv4 and IPv6 on the same network. |
| **Global unicast** | Routable IPv6 address type (2000::/3 range). |
| **ICMPv6** | IPv6 control messages; includes functions similar to ICMPv4 and NDP. |
| **IPv6** | 128-bit Internet Protocol; written in hex with `::` compression. |
| **Link-local** | `fe80::/10`; used on-link without global routing. |
| **NDP** | Neighbor Discovery Protocol; replaces ARP in IPv6 (NS/NA, RS/RA). |
| **SLAAC** | Stateless Address Autoconfiguration; host builds address from prefix in RA. |
| **Unique local** | `fc00::/7`; private IPv6 similar to RFC 1918. |

---

## Routing (Layer 3)

| Term | Definition |
| ---- | ---------- |
| **Administrative distance (AD)** | Trust ranking of route sources (lower preferred); static default 1, OSPF 110. |
| **ASBR** | Autonomous System Boundary Router; OSPF router injecting external routes. |
| **Convergence** | When all routers agree on a stable routing table after a change. |
| **Default gateway** | See **Gateway**. |
| **Distance vector** | Routing protocol class using hop count/metric from neighbors (RIP; legacy). |
| **DR / BDR** | Designated Router / Backup DR; OSPF reduces adjacencies on multi-access segments. |
| **Dynamic routing** | Routes learned automatically (OSPF, EIGRP, BGP). |
| **EIGRP** | Enhanced Interior Gateway Routing Protocol; Cisco advanced distance vector/hybrid. |
| **Floating static** | Backup static route with higher AD, used if primary fails. |
| **IGP** | Interior Gateway Protocol; routing within an organization (OSPF, EIGRP). |
| **Link-state** | Routing protocol class with full topology database (OSPF, IS-IS). |
| **Metric** | Value used to choose best path among routes to same destination. |
| **Next hop** | Next router IP along the path to a destination. |
| **OSPF** | Open Shortest Path First; link-state IGP using cost metric. |
| **Route summarization** | Advertising one aggregate prefix for multiple subnets. |
| **Routing table** | List of destinations, masks, next hops, and egress interfaces. |
| **Static route** | Manually configured route (`ip route`). |

---

## Inter-VLAN & WAN

| Term | Definition |
| ---- | ---------- |
| **Inter-VLAN routing** | Forwarding between VLANs at Layer 3 (router-on-a-stick or L3 switch SVI). |
| **MPLS** | Multiprotocol Label Switching; carrier WAN technology (awareness level for CCNA). |
| **Point-to-point link** | Single remote endpoint; common /30 or /31 addressing. |
| **Router-on-a-stick** | One router interface with subinterfaces for multiple VLANs over a trunk. |
| **WAN** | Wide Area Network; connects sites over long distance (leased line, Internet VPN, etc.). |

---

## Transport Layer (TCP/UDP)

| Term | Definition |
| ---- | ---------- |
| **Connection-oriented** | TCP establishes session before data transfer (handshake). |
| **Connectionless** | UDP sends without setup (best effort). |
| **Ephemeral port** | Temporary high-numbered source port for outbound sessions. |
| **FIN** | TCP flag to close a connection. |
| **Port** | 16-bit number identifying application/service on a host (0–65535). |
| **Socket** | Combination of IP, port, and protocol identifying an endpoint conversation. |
| **SYN** | TCP synchronize flag; starts three-way handshake. |
| **TCP** | Transmission Control Protocol; reliable, ordered delivery. |
| **Three-way handshake** | SYN, SYN-ACK, ACK sequence establishing TCP session. |
| **UDP** | User Datagram Protocol; low overhead, no guaranteed delivery. |
| **Window size** | TCP flow control; amount of unacknowledged data allowed in flight. |

---

## Application Layer & IP Services

| Term | Definition |
| ---- | ---------- |
| **DHCP** | Dynamic Host Configuration Protocol; assigns IP, mask, gateway, DNS automatically. |
| **DORA** | DHCP Discover, Offer, Request, Acknowledge process. |
| **DNS** | Domain Name System; resolves names to IP addresses. |
| **FTP** | File Transfer Protocol; cleartext file transfer (ports 20/21). |
| **HTTP / HTTPS** | Hypertext Transfer Protocol (Secure); web traffic ports 80/443. |
| **ip helper-address** | Cisco command to relay DHCP (and other UDP) broadcasts to a remote server. |
| **NAT** | Network Address Translation; maps private to public addresses. |
| **NTP** | Network Time Protocol; synchronizes device clocks. |
| **PAT** | Port Address Translation; NAT overload sharing one public IP using ports. |
| **SMTP** | Simple Mail Transfer Protocol; email transfer (port 25). |
| **SNMP** | Simple Network Management Protocol; monitoring and traps (v2c/v3). |
| **SSH** | Secure Shell; encrypted remote CLI (port 22). |
| **Syslog** | Standard for sending log messages to a central server. |
| **Telnet** | Cleartext remote login (port 23); insecure. |
| **TFTP** | Trivial FTP; simple UDP file transfer (often IOS upgrades). |

---

## Security

| Term | Definition |
| ---- | ---------- |
| **AAA** | Authentication, Authorization, and Accounting; security framework (RADIUS, TACACS+). |
| **BPDU Guard** | Disables port if unexpected BPDU received (protect access ports). |
| **CIA triad** | Confidentiality, Integrity, Availability. |
| **DMZ** | Demilitarized zone; semi-trusted segment for public servers. |
| **Firewall** | Device filtering traffic by rules (stateful inspection common). |
| **IDS / IPS** | Intrusion Detection / Prevention System; detect or block attacks. |
| **Malware** | Malicious software (virus, worm, ransomware). |
| **Phishing** | Social engineering via fake messages to steal credentials. |
| **Port security violation** | Switch action when unauthorized MAC appears (restrict, shutdown). |
| **RADIUS** | Remote Authentication Dial-In User Service; common for network access control. |
| **Social engineering** | Manipulating people to bypass security controls. |
| **Spoofing** | Falsifying source address or identity (IP, MAC, email). |
| **TACACS+** | Cisco-oriented AAA protocol; separates authentication and authorization. |
| **VPN** | Virtual Private Network; encrypted tunnel over untrusted network. |
| **WAF** | Web Application Firewall; protects HTTP/S applications. |
| **Zero Trust** | Security model assuming breach; verify every access continuously. |

---

## Wireless

| Term | Definition |
| ---- | ---------- |
| **802.11** | IEEE wireless LAN standards (Wi-Fi). |
| **AP** | Access Point; connects wireless clients to wired LAN. |
| **BSSID** | Basic Service Set Identifier; MAC of the AP radio. |
| **SSID** | Service Set Identifier; wireless network name. |
| **WLC** | Wireless LAN Controller; central management for lightweight APs. |
| **WPA2 / WPA3** | Wi-Fi security protocols; prefer WPA3 or WPA2-Enterprise in production. |

---

## High Availability & Redundancy

| Term | Definition |
| ---- | ---------- |
| **FHRP** | First Hop Redundancy Protocol; virtual default gateway (HSRP, VRRP, GLBP). |
| **GLBP** | Gateway Load Balancing Protocol; Cisco FHRP with load sharing. |
| **HSRP** | Hot Standby Router Protocol; Cisco FHRP with active/standby router. |
| **Preemption** | FHRP allows higher-priority router to reclaim active role when it returns. |
| **VRRP** | Virtual Router Redundancy Protocol; standards-based FHRP. |

---

## QoS (Awareness)

| Term | Definition |
| ---- | ---------- |
| **Classification** | Identifying traffic types for policy (ACL, marking). |
| **CoS** | Class of Service; 3-bit marking in 802.1Q tag (legacy LAN QoS). |
| **DSCP** | Differentiated Services Code Point; 6-bit marking in IP header. |
| **Policing** | Dropping or remarking traffic exceeding a rate limit. |
| **Queuing** | Buffering and scheduling packets when congestion occurs. |
| **Shaping** | Smoothing traffic to a configured rate over time. |

---

## Automation & Programmability (CCNA)

| Term | Definition |
| ---- | ---------- |
| **Ansible** | Automation tool using playbooks (YAML) for configuration management. |
| **API** | See General section; REST APIs common on controllers. |
| **Controller-based networking** | Centralized management (e.g., DNA Center concept). |
| **JSON** | JavaScript Object Notation; common API data format. |
| **REST** | Representational State Transfer; HTTP-based API style (GET, POST, etc.). |
| **SDN** | Software-Defined Networking; separates control and data planes conceptually. |
| **YAML** | Human-readable data format for configs and automation. |

---

## Active Directory (From This Repo)

| Term | Definition |
| ---- | ---------- |
| **AD DS** | Active Directory Domain Services; on-premises Microsoft directory. |
| **Domain** | Security and administrative boundary in Active Directory. |
| **Domain controller (DC)** | Server hosting writable/readable AD and Kerberos services. |
| **Entra ID** | Microsoft cloud identity service (formerly Azure AD). |
| **Forest** | Top-level AD container; one schema per forest. |
| **GPO** | Group Policy Object; settings applied to users and computers. |
| **Kerberos** | Ticket-based authentication used by AD domains. |
| **LDAP** | Lightweight Directory Access Protocol; directory query protocol. |
| **OU** | Organizational Unit; container for delegation and Group Policy. |
| **UPN** | User Principal Name; login form like `user@domain.com`. |

---

## Cisco IOS & Troubleshooting

| Term | Definition |
| ---- | ---------- |
| **CLI** | Command-Line Interface; router/switch configuration mode. |
| **Config register** | Boot behavior setting (password recovery context). |
| **Enable mode** | Privileged EXEC (`#`); show and debug commands. |
| **EXEC mode** | User mode (`>`); limited commands. |
| **Global configuration** | `configure terminal` mode for system-wide changes. |
| **Interface configuration** | Submode for specific interface settings. |
| **Running-config** | Active configuration in RAM. |
| **Startup-config** | Saved configuration in NVRAM (loads on boot). |
| **show command** | Display operational status (does not change config). |
| **Simlet** | Packet Tracer or exam simulation item requiring multi-step config. |

---

## Abbreviations Quick List

| Abbr. | Meaning |
| ----- | ------- |
| AD | Administrative Distance or Active Directory (context matters) |
| AP | Access Point |
| ARP | Address Resolution Protocol |
| BGP | Border Gateway Protocol |
| CLI | Command-Line Interface |
| DHCP | Dynamic Host Configuration Protocol |
| DNS | Domain Name System |
| FHRP | First Hop Redundancy Protocol |
| FTP | File Transfer Protocol |
| HTTP(S) | Hypertext Transfer Protocol (Secure) |
| ICMP | Internet Control Message Protocol |
| IOS | Cisco Internetwork Operating System |
| IP | Internet Protocol |
| LAN | Local Area Network |
| MAC | Media Access Control |
| NAT | Network Address Translation |
| OSPF | Open Shortest Path First |
| PAT | Port Address Translation |
| PDU | Protocol Data Unit |
| QoS | Quality of Service |
| RIP | Routing Information Protocol |
| SNMP | Simple Network Management Protocol |
| SSH | Secure Shell |
| STP | Spanning Tree Protocol |
| TCP | Transmission Control Protocol |
| UDP | User Datagram Protocol |
| VLAN | Virtual LAN |
| VPN | Virtual Private Network |
| WAN | Wide Area Network |
| WLAN | Wireless LAN |

---

## Related Study Files

| Topic | File |
| ----- | ---- |
| Full CCNA syllabus | [CCNA-Networking-Study-Guide.md](CCNA-Networking-Study-Guide.md) |
| TCP/IP model | [TCP_IP-model/TCP-IP-Model-Study-Guide.md](TCP_IP-model/TCP-IP-Model-Study-Guide.md) |
| Binary and hex | [binary-hexadecimal/Binary-and-Hex-for-Networking.md](binary-hexadecimal/Binary-and-Hex-for-Networking.md) |
| Subnetting | [IP-addresses/subnetting.md](IP-addresses/subnetting.md) |
| Commands | [cheat-sheets/command-cheat-sheet.md](cheat-sheets/command-cheat-sheet.md) |
| Active Directory | [active-directory/Active-Directory-Study-Guide.md](active-directory/Active-Directory-Study-Guide.md) |
| Labs | [labs/](labs/) |

---

*Add your own terms in a notebook as you encounter them in Packet Tracer and practice exams.*
