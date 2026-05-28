# Inter-VLAN Routing

## What is Inter-VLAN Routing?

Inter-VLAN routing is a networking technique used to allow communication between different VLANs (Virtual Local Area Networks).

By default, devices in one VLAN cannot communicate with devices in another VLAN because VLANs are designed to separate and isolate network traffic.

Inter-VLAN routing solves this problem by using a Layer 3 device, such as:

- A router
- A Layer 3 switch

---

# Why is it Important?

VLANs improve:

- Security
- Organization
- Network performance

However, departments or systems in different VLANs may still need to communicate.

Example:

- HR VLAN needs access to a server in another VLAN
- Sales VLAN needs internet access through a router

Inter-VLAN routing allows these networks to communicate safely and efficiently.

---

# Simple Analogy

Think of VLANs like separate rooms in a building.

- Devices in the same room can talk directly
- Devices in different rooms need a doorway

The router or Layer 3 switch acts as the doorway between VLANs.

---

# How Inter-VLAN Routing Works

1. A device sends traffic to its default gateway
2. The router or Layer 3 switch receives the packet
3. The device determines the destination VLAN
4. The packet is routed to the correct VLAN
5. The destination device receives the data

---

# Example Network

## VLAN 10 - Sales

Network: 192.168.10.0/24

Devices:

- PC1 → 192.168.10.10
- PC2 → 192.168.10.11

Gateway:

- 192.168.10.1

---

## VLAN 20 - HR

Network: 192.168.20.0/24

Devices:

- PC3 → 192.168.20.10
- PC4 → 192.168.20.11

Gateway:

- 192.168.20.1

---

# Communication Example

If PC1 in VLAN 10 wants to communicate with PC3 in VLAN 20:

1. PC1 sends the packet to its gateway (192.168.10.1)
2. The router receives the packet
3. The router routes the packet to VLAN 20
4. PC3 receives the packet

Without inter-VLAN routing, this communication would fail.

---

# Methods of Inter-VLAN Routing

## 1. Router-on-a-Stick

This method uses:

- One physical router interface
- Multiple subinterfaces
- A trunk connection to the switch

Each subinterface is assigned to a VLAN.

Example:

- G0/0.10 → VLAN 10
- G0/0.20 → VLAN 20

Advantages:

- Simple for learning
- Good for small networks

Disadvantages:

- Slower than Layer 3 switching
- Router can become a bottleneck

---

## 2. Layer 3 Switch Routing

A multilayer switch can perform routing internally using:

- SVIs (Switched Virtual Interfaces)

Example:

- Interface VLAN 10
- Interface VLAN 20

Advantages:

- Faster
- More scalable
- Better for enterprise networks

Disadvantages:

- More expensive hardware

---

# Key Networking Concepts

## VLAN

A VLAN is a logical separation of a network on a switch.

## Trunk Port

A trunk port carries traffic for multiple VLANs between devices.

## Access Port

An access port belongs to one VLAN only.

## Default Gateway

The IP address devices use to communicate outside their own VLAN.

## Layer 3 Device

A device capable of routing traffic between networks.

---

# Summary

Inter-VLAN routing allows devices in different VLANs to communicate using a router or Layer 3 switch.

Important ideas:

- VLANs isolate traffic
- Routers or Layer 3 switches connect VLANs
- Default gateways are required
- Trunk links carry multiple VLANs
- Inter-VLAN routing is essential in real-world networks

This concept is heavily used in enterprise networking and is important for Cisco CCNA studies.