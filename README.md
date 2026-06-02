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

### 2026-06-02 — What `Fa0/23` means

**What I learned:** On a Cisco switch, **`Fa0/23`** is interface notation: **`Fa`** = FastEthernet, **`0`** = module/slot (on a 2960-style switch this is usually fixed at 0), **`23`** = port number. So it is **port 23** on the FastEthernet module — not “port 0 and port 23.” In the [Small Office Network Lab](labs/CCNA_Small_Office_Network_Lab.md), **SW1 Fa0/23** connects to **SW2 Fa0/23** as the **trunk** between switches.

**What clicked:** The slash separates **where** on the device (slot) from **which port** on that module.

**Still fuzzy:** When labs use `Gig0/0` on a router vs `Fa0/24` on a switch — same idea, different interface type prefix.

**Next steps:** Match port labels on the topology diagram to `interface fa0/23` in config without mixing up access ports (PCs) and trunk ports.

---

### Example entry (delete or replace when I write my own)

### 2026-01-01 — First VLAN lab

**What I learned:** VLANs split one switch into separate broadcast domains. Access ports belong to one VLAN; trunks carry multiple VLANs with 802.1Q tags.

**What clicked:** Without a router or L3 switch, VLAN 10 and VLAN 20 cannot talk to each other — same switch, different L2 domains.

**Still fuzzy:** Native VLAN mismatches on trunks and when I need `switchport trunk native vlan` vs leaving default.

**Next steps:** Finish [Small Office Network Lab](labs/CCNA_Small_Office_Network_Lab.md); practice `show vlan brief` and `show interfaces trunk`.

---

*Last updated: 6/2*