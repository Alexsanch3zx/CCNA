# Cisco Networking Command Cheat Sheet

**Audience:** CCNA (200-301) study and lab work in Cisco IOS / Packet Tracer.

Quick reference for configuration and verification. Syntax may vary slightly by platform (router vs switch, IOS version).

---

## IOS Mode Navigation


| Prompt             | Mode             | Enter                | Exit               |
| ------------------ | ---------------- | -------------------- | ------------------ |
| `>`                | User EXEC        | —                    | `logout` / `exit`  |
| `#`                | Privileged EXEC  | `enable`             | `disable` / `exit` |
| `(config)#`        | Global config    | `configure terminal` | `exit` / `end`     |
| `(config-if)#`     | Interface config | `interface g0/0`     | `exit`             |
| `(config-line)#`   | Line config      | `line console 0`     | `exit`             |
| `(config-router)#` | Routing protocol | `router ospf 1`      | `exit`             |


**Shortcuts:** `Ctrl+Z` or `end` → privileged EXEC. `Tab` = autocomplete. `?` = help. `show running-config` often works from privileged EXEC only.

---

## Initial Device Setup

```cisco
enable
configure terminal
hostname R1

enable secret YourSecretPassword
service password-encryption

banner motd # Authorized access only #

line console 0
 password cisco
 login

line vty 0 4
 password cisco
 login
 transport input ssh

end
write memory
```

**Save config:** `copy running-config startup-config` or `write memory` (`wr`).

**Erase startup (lab reset):** `write erase` → reload.

---

## Interface Configuration

```cisco
interface g0/0
 description LAN-to-Switch
 ip address 192.168.1.1 255.255.255.0
 no shutdown
```

**Subinterface (router-on-a-stick):**

```cisco
interface g0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
```

**Serial (DCE sets clock):**

```cisco
interface s0/0/0
 ip address 172.16.0.1 255.255.255.252
 clock rate 128000
 no shutdown
```

---

## VLANs & Switchport Modes

**Create VLANs:**

```cisco
vlan 10
 name HR
vlan 20
 name SALES
```

**Access port (host):**

```cisco
interface fa0/1
 switchport mode access
 switchport access vlan 10
```

**Trunk port (switch-to-switch or switch-to-router):**

```cisco
interface fa0/24
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30
```

**Range syntax:**

```cisco
interface range fa0/1-12
 switchport mode access
 switchport access vlan 10
```

---

## Inter-VLAN Routing

### Router-on-a-Stick

1. Trunk on switch port to router.
2. Subinterface per VLAN on router (`encapsulation dot1Q <vlan-id>`).
3. Each subinterface = default gateway for that VLAN.

### Layer 3 Switch (SVI)

```cisco
ip routing

interface vlan 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
```

---

## EtherChannel (LACP)

```cisco
interface range gi1/0/23-24
 channel-group 1 mode active
 switchport mode trunk

interface port-channel 1
 switchport mode trunk
```


| Mode      | Behavior                     |
| --------- | ---------------------------- |
| `active`  | LACP — initiates negotiation |
| `passive` | LACP — responds only         |
| `on`      | Static EtherChannel, no LACP |


Verify: `show etherchannel summary`

---

## Spanning Tree (STP)

```cisco
spanning-tree mode rapid-pvst

spanning-tree vlan 10 root primary
spanning-tree vlan 10 root secondary

interface fa0/1
 spanning-tree portfast
 spanning-tree bpduguard enable
```


| Command                      | Purpose                    |
| ---------------------------- | -------------------------- |
| `show spanning-tree`         | Root bridge, roles, states |
| `show spanning-tree vlan 10` | Per-VLAN detail            |
| `show spanning-tree root`    | Root bridge summary        |


**Port states:** Blocking → Listening → Learning → Forwarding (classic STP).

---

## Static & Default Routes

```cisco
ip route 192.168.2.0 255.255.255.0 10.0.0.2
ip route 0.0.0.0 0.0.0.0 203.0.113.1
```

**Floating static (backup):** add higher AD — e.g. `ip route 0.0.0.0 0.0.0.0 10.0.0.2 250`


| Route code | Meaning       |
| ---------- | ------------- |
| `C`        | Connected     |
| `S`        | Static        |
| `O`        | OSPF          |
| `*`        | Default route |


---

## OSPFv2 (Single Area)

```cisco
router ospf 1
 router-id 1.1.1.1
 network 192.168.10.0 0.0.0.255 area 0
 network 172.16.0.0 0.0.0.3 area 0
 default-information originate
```

**Loopback (stable router ID):**

```cisco
interface loopback 0
 ip address 1.1.1.1 255.255.255.255
```


| Verify     | Command                        |
| ---------- | ------------------------------ |
| Neighbors  | `show ip ospf neighbor`        |
| Routes     | `show ip route ospf`           |
| Process    | `show ip protocols`            |
| Interfaces | `show ip ospf interface brief` |


**Wildcard mask cheat:** /24 = `0.0.0.255`, /30 = `0.0.0.3`, /32 = `0.0.0.0`

---

## DHCP

**On router (server):**

```cisco
ip dhcp excluded-address 192.168.10.1 192.168.10.10

ip dhcp pool HR
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 192.168.30.10
```

**Relay on L3 interface / SVI:**

```cisco
interface vlan 10
 ip helper-address 192.168.30.10
```

Verify: `show ip dhcp binding`, `show ip dhcp pool`

---

## NAT / PAT

```cisco
access-list 1 permit 192.168.10.0 0.0.0.255

interface g0/0
 ip nat inside

interface g0/1
 ip nat outside

ip nat inside source list 1 interface g0/1 overload
```

Verify: `show ip nat translations`, `show ip nat statistics`

---

## Access Control Lists (ACLs)

**Standard (source IP only, 1–99 / 1300–1999):**

```cisco
access-list 10 permit 192.168.10.0 0.0.0.255
access-list 10 deny any
```

**Extended (source, destination, protocol, ports, 100–199 / 2000–2699):**

```cisco
ip access-list extended BLOCK-SALES-TO-HR
 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
 permit ip any any
```

**Apply to interface:**

```cisco
interface g0/0.20
 ip access-group BLOCK-SALES-TO-HR in
```

**Rules:** Top-down processing; implicit **deny all** at end. Place specific permits before general denies.

Verify: `show access-lists` (check hit counts)

---

## Port Security

```cisco
interface fa0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
```

**Recover err-disabled port:**

```cisco
interface fa0/1
 shutdown
 no shutdown
```

Verify: `show port-security`, `show port-security interface fa0/1`

---

## SSH (Secure Remote Access)

```cisco
ip domain-name lab.local
crypto key generate rsa modulus 1024

username admin privilege 15 secret YourPassword

line vty 0 4
 login local
 transport input ssh

ip access-list standard MGMT
 permit 192.168.1.0 0.0.0.255
 deny any

line vty 0 4
 access-class MGMT in
```

Connect from PC: `ssh -l admin 192.168.1.1`

---

## IPv6 Essentials

```cisco
ipv6 unicast-routing

interface g0/0
 ipv6 address 2001:db8:10::1/64
 ipv6 enable
```

**SLAAC / DHCPv6 (conceptual):** `ipv6 address autoconfig` on hosts; router advertises via RA.

Verify: `show ipv6 interface brief`, `show ipv6 route`, `show ipv6 neighbors`

---

## NTP & Syslog (Basics)

```cisco
ntp server 192.168.30.10

logging host 192.168.30.10
logging on
```

Verify: `show clock`, `show ntp status`, `show logging`

---

## Essential `show` Commands

### Layer 2 / Switching


| Command                     | Shows                       |
| --------------------------- | --------------------------- |
| `show vlan brief`           | VLANs and port assignments  |
| `show interfaces trunk`     | Trunk ports, allowed VLANs  |
| `show mac address-table`    | MAC → port mapping          |
| `show spanning-tree`        | STP topology                |
| `show etherchannel summary` | Port-channel status         |
| `show port-security`        | Secure ports and violations |


### Layer 3 / Routing


| Command                   | Shows                            |
| ------------------------- | -------------------------------- |
| `show ip interface brief` | Interface IPs and status         |
| `show ip route`           | Routing table                    |
| `show ip protocols`       | Active routing protocols         |
| `show ip ospf neighbor`   | OSPF adjacencies                 |
| `show cdp neighbors`      | Directly connected Cisco devices |
| `show arp`                | IPv4 → MAC mappings              |


### Config & Troubleshooting


| Command                  | Shows                      |
| ------------------------ | -------------------------- |
| `show running-config`    | Active config in RAM       |
| `show startup-config`    | Saved config in NVRAM      |
| `show ip interface g0/0` | Interface detail, errors   |
| `show interfaces status` | Port up/down, VLAN, speed  |
| `show logging`           | System messages            |
| `show version`           | IOS version, uptime, model |


**Filter output:**

```cisco
show running-config | section ospf
show running-config | include vlan
show ip route | include 192.168
```

---

## Troubleshooting Commands

```cisco
ping 192.168.10.10
ping 192.168.10.10 source 192.168.20.1
traceroute 203.0.113.10

debug ip routing
debug ip ospf events
undebug all
```

**Clear counters / tables (lab use):**

```cisco
clear mac address-table dynamic
clear ip ospf process
reload
```

---

## CDP & LLDP

```cisco
cdp run
show cdp neighbors
show cdp neighbors detail

lldp run
show lldp neighbors
```

---

## Common Exam Pitfalls


| Mistake                          | Fix                                    |
| -------------------------------- | -------------------------------------- |
| Forgot `no shutdown`             | Interface stays administratively down  |
| Access port on trunk link        | Use `switchport mode trunk`            |
| RoS without trunk to router      | Switch uplink must be 802.1Q trunk     |
| ACL wrong direction              | `in` = into interface; `out` = leaving |
| NAT inside/outside swapped       | Check `show ip nat statistics`         |
| OSPF network statement wrong     | Match wildcard to interface subnet     |
| DHCP pool overlaps gateway       | Use `ip dhcp excluded-address`         |
| Port security without sticky MAC | Learn MAC first or use `sticky`        |


---

## Quick Lab Verification Checklist

1. `show ip interface brief` — all needed interfaces **up/up**
2. `show vlan brief` — correct VLAN → port map
3. `show interfaces trunk` — VLANs allowed on trunks
4. `ping` gateway from each subnet
5. `ping` across VLANs / sites
6. `show ip route` — expected routes present
7. `show access-lists` — ACL hits increment when testing
8. `show spanning-tree` — no unexpected blocking on critical paths

---

## Related Files in This Repo

- [CCNA-Networking-Study-Guide.md](CCNA-Networking-Study-Guide.md) — full syllabus
- [subnetting.md](IP-addresses/subnetting.md) — IPv4 math
- [inter_vlan_routing.md](inter_vlan_routing.md) — inter-VLAN concepts
- [DHCP_Notes.md](IP-addresses/DHCP_Notes.md) — DHCP deep dive
- [labs/](labs/) — Packet Tracer lab walkthroughs

