# DHCP (Dynamic Host Configuration Protocol)

## What Does DHCP Stand For?

**DHCP** stands for **Dynamic Host Configuration Protocol**.

Its job is to **automatically assign IP addresses and other network settings** to devices on a network.

Instead of manually configuring:

- IP Address (e.g., `192.168.1.10`)
- Subnet Mask (e.g., `255.255.255.0`)
- Default Gateway (e.g., `192.168.1.1`)
- DNS Server (e.g., `8.8.8.8`)

DHCP does it automatically when a device connects to the network.

---

## Example

### Without DHCP
You manually configure every PC.

### With DHCP
A DHCP server automatically assigns:

- PC1 → `192.168.1.10`
- PC2 → `192.168.1.11`
- PC3 → `192.168.1.12`

---

## In Cisco Packet Tracer

When an instruction says **"Configure DHCP"**, it usually means:

1. Create a DHCP server (often on a router or server).
2. Define a pool of available IP addresses.
3. Specify:
   - Network address
   - Subnet mask
   - Default gateway
   - DNS server
4. Set PCs to obtain an IP address automatically.

---

## DHCP Process (DORA)

A common CCNA topic:

1. **Discover** – Client asks, "Any DHCP servers?"
2. **Offer** – Server offers an IP address.
3. **Request** – Client requests that IP.
4. **Acknowledge** – Server confirms the assignment.

### Memory Trick

**DORA** = **Discover → Offer → Request → Acknowledge**

---

## Why DHCP Is Important

DHCP is one of the first network services you'll configure after learning:

- IP Addressing
- Subnetting
- VLANs
- Inter-VLAN Routing
- Routers and Switches

It makes network management much easier by automatically assigning network settings to devices.
