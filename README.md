# CCNA Study Guide & Learning Journal

Personal repository for **Cisco CCNA (200-301)** prep, Packet Tracer labs, and notes as I move into tech.

The study guides here are reference material. **This README is my reflection journal** — I add entries over time about what I actually understood, what confused me, and what I still need to practice.

---

## Study Materials (Repo Map)

| Folder / file | What's inside |
| ------------- | ------------- |
| [CCNA-Networking-Study-Guide.md](CCNA-Networking-Study-Guide.md) | Full CCNA-aligned syllabus |
| [terminology.md](terminology.md) | Glossary of networking terms |
| [TCP_IP-model/](TCP_IP-model/) | TCP/IP model deep dive |
| [binary-hexadecimal/](binary-hexadecimal/) | Binary and hex for networking |
| [IP-addresses/](IP-addresses/) | Subnetting, DHCP, IPv4 vs IPv6 |
| [cheat-sheets/](cheat-sheets/) | IOS command cheat sheets |
| [labs/](labs/) | Packet Tracer labs and project ideas |
| [networking-basics.md](networking-basics.md) | Networking basics (cybersecurity angle) |
| [inter_vlan_routing.md](inter_vlan_routing.md) | Inter-VLAN routing notes |
| [configuration_explained.md](configuration_explained.md) | CLI configuration walkthroughs |
| [active-directory/](active-directory/) | Active Directory study guide |
| [png/](png/) | Diagrams and images |

**Exam focus:** Cisco **CCNA 200-301**

---

## How I Use the Reflection Section

When I finish a chapter, lab, or study session, I add an entry below. I try to answer:

1. **What did I learn?** (in my own words, not copy-paste from the guide)  
2. **What clicked?** (the "aha" moment)  
3. **What is still fuzzy?** (be honest — this is where I study next)  
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

*Add newest entries at the top of this section.*

---

### YYYY-MM-DD — [Title]

**What I learned:**



**What clicked:**



**Still fuzzy:**



**Next steps:**



---

### Example entry (delete or replace when I write my own)

### 2026-01-01 — First VLAN lab

**What I learned:** VLANs split one switch into separate broadcast domains. Access ports belong to one VLAN; trunks carry multiple VLANs with 802.1Q tags.

**What clicked:** Without a router or L3 switch, VLAN 10 and VLAN 20 cannot talk to each other — same switch, different L2 domains.

**Still fuzzy:** Native VLAN mismatches on trunks and when I need `switchport trunk native vlan` vs leaving default.

**Next steps:** Finish [Small Office Network Lab](labs/CCNA_Small_Office_Network_Lab.md); practice `show vlan brief` and `show interfaces trunk`.

---

## Progress Tracker (optional)

Check off topics as I feel exam-ready (not just "I read it once").

- [ ] OSI / TCP/IP models and encapsulation  
- [ ] Binary, hex, and subnetting (can do VLSM by hand)  
- [ ] Switching: VLANs, trunks, STP basics  
- [ ] Inter-VLAN routing (router-on-a-stick and SVI)  
- [ ] IPv4 / IPv6 addressing  
- [ ] Static routing and OSPF single-area  
- [ ] DHCP, DNS, NAT/PAT  
- [ ] ACLs and port security  
- [ ] Wireless fundamentals  
- [ ] Automation / APIs (CCNA-level awareness)  
- [ ] Completed at least 3 Packet Tracer labs from [labs/](labs/)  

---

## Links

- **Remote repo:** https://github.com/Alexsanch3zx/CCNA.git  
- **Cisco exam topics:** verify current [200-301 exam topics](https://www.cisco.com/c/en/us/training-events/training-certifications/exams/current-list/ccna-200-301.html) before scheduling  

---

*Last updated: edit this line when I add a reflection entry.*
