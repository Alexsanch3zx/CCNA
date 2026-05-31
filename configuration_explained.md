# What Does 'Configuring' Mean in Networking?

## Definition

In networking, **configuring** means **setting up or changing the settings of a network device so it behaves the way you want**.

Think of it like configuring a new phone:

- Setting the Wi-Fi network
- Changing the wallpaper
- Creating a passcode

In networking, you're doing the same thing, but with routers, switches, firewalls, servers, and PCs.

---

## Examples of Network Configuration

### 1. Configuring an IP Address

You tell a device what IP address to use.

```cisco
interface GigabitEthernet0/0
ip address 192.168.1.1 255.255.255.0
no shutdown
```

This configures:

- IP Address: 192.168.1.1
- Subnet Mask: 255.255.255.0
- Turns the interface on

---

### 2. Configuring a Switch Port

You tell a switch which VLAN a port belongs to.

```cisco
interface FastEthernet0/1
switchport mode access
switchport access vlan 10
```

This configures:

- Port Fa0/1
- Access mode
- VLAN 10

---

### 3. Configuring DHCP

You tell a router how to automatically assign IP addresses.

```cisco
ip dhcp pool OFFICE
network 192.168.1.0 255.255.255.0
default-router 192.168.1.1
```

This configures:

- A DHCP pool
- The network range
- The default gateway

---

### 4. Configuring Routing

You tell a router where to send packets.

```cisco
ip route 0.0.0.0 0.0.0.0 10.0.0.1
```

This configures:

- A default route
- The next-hop router

---

## In CCNA Terms

When you hear:

> Configure the router

it usually means:

- Assign IP addresses
- Enable interfaces (`no shutdown`)
- Create VLANs
- Configure DHCP
- Configure routing
- Configure security settings
- Configure passwords

You're essentially **telling the device how to operate on the network**.

---

## Easy Way to Remember

- **Monitoring** = Looking at the network
- **Troubleshooting** = Fixing the network
- **Configuring** = Setting up the network

When you're in Cisco Packet Tracer typing commands into a router or switch, you're performing network configuration.
