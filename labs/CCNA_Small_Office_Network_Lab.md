# CCNA Packet Tracer Lab – Small Office Network

## Lab Overview

This lab expands a basic network of 4 PCs connected to a switch into a small business network that introduces:

- VLANs
- Trunking
- Inter-VLAN Routing
- DHCP
- DNS
- HTTP Services
- ACLs
- Port Security
- Spanning Tree
- Troubleshooting

---

# Network Topology

```text
                 Router
                    |
               Switch 1
              /    |    \
            PC0   PC1   Switch 2
                        /   |   \
                      PC2  PC3  Server
                               \
                               Printer
```

---

# Devices Needed

- 1 Router (2911)
- 2 Switches (2960)
- 4 PCs
- 1 Server
- 1 Printer

---

# VLAN Design


| VLAN | Name     | Network         |
| ---- | -------- | --------------- |
| 10   | HR       | 192.168.10.0/24 |
| 20   | Sales    | 192.168.20.0/24 |
| 30   | Servers  | 192.168.30.0/24 |
| 40   | Printers | 192.168.40.0/24 |


---

# IP Addressing

## HR


| Device | IP Address    |
| ------ | ------------- |
| PC0    | 192.168.10.10 |
| PC1    | 192.168.10.11 |


Gateway: 192.168.10.1

## Sales


| Device | IP Address    |
| ------ | ------------- |
| PC2    | 192.168.20.10 |
| PC3    | 192.168.20.11 |


Gateway: 192.168.20.1

## Server


| Device  | IP Address    |
| ------- | ------------- |
| Server0 | 192.168.30.10 |


Gateway: 192.168.30.1

## Printer


| Device   | IP Address    |
| -------- | ------------- |
| Printer0 | 192.168.40.10 |


Gateway: 192.168.40.1

---

# Step 1: Create VLANs

```cisco
enable
configure terminal

vlan 10
name HR

vlan 20
name SALES

vlan 30
name SERVERS

vlan 40
name PRINTERS
```

---

# Step 2: Assign Access Ports

Example:

```cisco
interface range fa0/1-2
switchport mode access
switchport access vlan 10
```

```cisco
interface range fa0/3-4
switchport mode access
switchport access vlan 20
```

Assign the Server and Printer ports to VLAN 30 and VLAN 40 respectively.

---

# Step 3: Configure Switch-to-Switch Trunk

Connect:

Switch1 Fa0/23 <-> Switch2 Fa0/23

Configure both switches:

```cisco
interface fa0/23
switchport mode trunk
```

Verify:

```cisco
show interfaces trunk
```

---

# Step 4: Configure Router-on-a-Stick

```cisco
enable
configure terminal

interface g0/0
no shutdown
```

## VLAN 10

```cisco
interface g0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
```

## VLAN 20

```cisco
interface g0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
```

## VLAN 30

```cisco
interface g0/0.30
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.0
```

## VLAN 40

```cisco
interface g0/0.40
encapsulation dot1Q 40
ip address 192.168.40.1 255.255.255.0
```

Verify:

```cisco
show ip interface brief
```

---

# Step 5: Configure DHCP

```cisco
ip dhcp pool HR
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1
```

Create additional pools for:

- Sales
- Servers
- Printers

Set PCs to DHCP and verify they receive addresses automatically.

---

# Step 6: Configure DNS

On the Server:

Services → DNS → On

Add:

```text
company.local -> 192.168.30.10
```

Test:

```bash
ping company.local
```

---

# Step 7: Configure HTTP Service

On Server:

Services → HTTP → On

Example homepage:

```html
Welcome to Company Network
```

Test from PCs:

```text
http://192.168.30.10
```

or

```text
http://company.local
```

---

# Step 8: Configure ACL Security

Block Sales from accessing HR:

```cisco
access-list 100 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
access-list 100 permit ip any any
```

Apply:

```cisco
interface g0/0.20
ip access-group 100 in
```

Verify that Sales devices can no longer communicate with HR devices.

---

# Step 9: Configure Port Security

```cisco
interface fa0/1

switchport port-security
switchport port-security maximum 1
switchport port-security violation shutdown
```

Test by connecting a different device.

---

# Step 10: Investigate Spanning Tree

```cisco
show spanning-tree
```

Identify:

- Root Bridge
- Forwarding Ports
- Blocking Ports

---

# Verification Commands

## Switch

```cisco
show vlan brief
show interfaces trunk
show running-config
show spanning-tree
```

## Router

```cisco
show ip interface brief
show running-config
show ip route
```

---

# Troubleshooting Challenges

## Scenario 1

Incorrect default gateway.

Find and fix the issue.

## Scenario 2

Incorrect VLAN assignment.

Find and fix the issue.

## Scenario 3

Trunk port disabled.

Find and fix the issue.

## Scenario 4

ACL blocking traffic.

Find and fix the issue.

---

# Skills Learned

- VLAN Configuration
- Trunking
- Router-on-a-Stick
- Inter-VLAN Routing
- DHCP
- DNS
- HTTP Services
- ACLs
- Port Security
- Spanning Tree
- Troubleshooting

This lab closely resembles a small enterprise network and covers many core CCNA topics.