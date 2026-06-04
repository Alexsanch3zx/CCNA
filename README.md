# CCNA Study Guide & Learning Journal

Personal repository for **Cisco CCNA (200-301)** prep, Packet Tracer labs, and notes as I move into tech.

**This README is my reflection journal** — I add entries over time about what I actually understood, what confused me, and what I still need to practice.

---

## How I Use the Reflection Section

When I finish a chapter, lab, or study session, I add an entry below. I try to answer:

1. **What did I learn?**
2. **What clicked?**
3. **What is still fuzzy?**
4. **What will I lab or review next?**

Short entries are fine. Consistency matters more than length.

### Copy-paste template for a new entry

```markdown
### YYYY-MM-DD — [Topic or lab name]

**What I learned:**

**What clicked:**

**Still fuzzy:**

**Next steps:**
```

---

## Reflections

### 2026-06-04 — Encapsulation (data going down the stack)

**What I learned:** **Encapsulation** is how data gets wrapped layer by layer **on the way out**. The application sends data; each lower layer adds its own header (TCP/UDP, then IP, then Ethernet frame with MAC addresses, then bits on the wire). On the way **in**, the device **de-encapsulates** — strips headers from the bottom up. A **switch** cares about the **frame (MAC)**; a **router** looks at the **packet (IP)** and builds a **new frame** for the next hop.

**What clicked:** When I ping another PC on the same VLAN, the IP addresses stay the same on that hop, but if the packet crosses a **router**, the **MAC addresses change** every time — IP usually does not (without NAT). Packet Tracer **Simulation** mode lets me watch this happen step by step.

**Still fuzzy:** Remembering the exact **PDU names** (segment vs packet vs frame) without mixing them up on exam questions.

**Next steps:** Trace one **same-VLAN** ping and one **cross-VLAN** ping in Simulation; review [TCP-IP-Model-Study-Guide.md](TCP_IP-model/TCP-IP-Model-Study-Guide.md) Part 4.

---

### 2026-06-03 — Checking VLANs (CLI and GUI)

**What I learned:** I can verify VLANs with `show vlan brief` in the switch CLI, or click the switch in Packet Tracer → **Config** → **VLAN Database** to see the VLANs I created without typing commands.

**What clicked:** CLI and GUI show the same info — good for lab checks when I forget a command.

---

### 2026-06-03 — IPv4 is 32 bits and `/24` math (`2^n` and `2^n - 2`)

**What I learned:** Every **IPv4 address is 32 bits** total, written as four octets (e.g. `192.168.10.0`). The `**/24`** in `192.168.10.0/24` means **24 bits are network** and the rest are **host** bits. So: **32 - 24 = 8 host bits**. From there:


| Calculation                         | Formula               | For `/24` (8 host bits) |
| ----------------------------------- | --------------------- | ----------------------- |
| Total addresses in the subnet       | `2^n` (n = host bits) | `2^8` = **256**         |
| Usable host addresses (typical LAN) | `2^n - 2`             | `2^8 - 2` = **254**     |


The **-2** is because the **network address** (host bits all 0) and **broadcast** (host bits all 1) are not assigned to normal hosts.

**What clicked:** The slash number is not “how many hosts” — it is **how many network bits**. I subtract from **32** first, then use **n** in `2^n`.

**Still fuzzy:** Doing the same math when the subnet boundary is not in the last octet (e.g. `/22` or `/26) — I need to know which octet the block size lands in.

**Next steps:** Practice three random prefixes (`/26`, `/27`, `/30`) on paper; use [subnetting.md](IP-addresses/subnetting.md) for VLSM next.

---

### 2026-06-02 — DHCP (Dynamic Host Configuration Protocol)

**What I learned:** **DHCP** assigns IP settings automatically (address, mask, default gateway, DNS) so I do not have to type them on every PC. The client process is **DORA**: **D**iscover, **O**ffer, **R**equest, **A**cknowledge. On the router in my lab, I create **pools** per VLAN (e.g. HR `192.168.10.0`) and set **default-router** to the subinterface IP (`.1`). `**ip dhcp excluded-address`** keeps specific IPs free for static devices (server, printer). PCs in Packet Tracer use **DHCP** in Desktop → IP Configuration instead of static.

**What clicked:** The **default gateway** in the pool must be the router interface for that VLAN — otherwise PCs get an IP but cannot reach other subnets. DHCP is an **Application layer** service, but it only works if **L2/L3** (VLANs, routing, trunks) are already correct.

**Still fuzzy:** When the DHCP server is on a **different subnet** than the clients — I need `**ip helper-address`** on the router to relay broadcasts (not needed when R1 is the server and default gateway for each VLAN in my small office lab).

**Next steps:** Run `show ip dhcp binding` on R1 after PCs lease; re-read [DHCP_Notes.md](IP-addresses/DHCP_Notes.md); finish DHCP step in [Small Office Network Lab](labs/CCNA_Small_Office_Network_Lab.md).

---

### 2026-06-02 — What `Fa0/23` means

**What I learned:** On a Cisco switch, `**Fa0/23`** is interface notation: `**Fa`** = FastEthernet, `**0**` = module/slot (on a 2960-style switch this is usually fixed at 0), `**23**` = port number. So it is **port 23** on the FastEthernet module — not “port 0 and port 23.” In the [Small Office Network Lab](labs/CCNA_Small_Office_Network_Lab.md), **SW1 Fa0/23** connects to **SW2 Fa0/23** as the **trunk** between switches.

**What clicked:** The slash separates **where** on the device (slot) from **which port** on that module.

**Still fuzzy:** When labs use `Gig0/0` on a router vs `Fa0/24` on a switch — same idea, different interface type prefix.

**Next steps:** Match port labels on the topology diagram to `interface fa0/23` in config without mixing up access ports (PCs) and trunk ports.

---

### 2026-06-01 — What `VLANs` are

### 2026-01-01 — First VLAN lab

**What I learned:** VLANs split one switch into separate broadcast domains. Access ports belong to one VLAN; trunks carry multiple VLANs with 802.1Q tags.

**What clicked:** Without a router or L3 switch, VLAN 10 and VLAN 20 cannot talk to each other — same switch, different L2 domains.

**Still fuzzy:** Native VLAN mismatches on trunks and when I need `switchport trunk native vlan` vs leaving default.

**Next steps:** Finish [Small Office Network Lab](labs/CCNA_Small_Office_Network_Lab.md); practice `show vlan brief` and `show interfaces trunk`.

---

*Last updated: 6/2*