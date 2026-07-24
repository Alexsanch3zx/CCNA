# CCNA Packet Tracer Lab – ACL & Port Security (Mini)

## Lab Overview

A **short** lab that focuses on two CCNA security tools that sit at different layers:

| Feature | Where | What it controls |
| ------- | ----- | ---------------- |
| **Port security** | Switch access port | Which / how many **MACs** may use that port |
| **ACL** | Router interface | Which **IP** traffic is permitted or denied |

You will build a tiny three-VLAN LAN, lock one user port with port security, then apply an extended ACL so Guests cannot reach the server while Staff still can.

**Prerequisite:** VLANs, trunks, and router-on-a-stick (Small Office or ACL labs).

**Theory pair:** [ACL-Study-Guide.md](../../ACL-Study-Guide.md) · port security notes in [MAC-addresses.md](../../MAC-addresses.md)

**Why this lab exists:** The ACL lab teaches filtering without port security. The Layer 2 Security lab covers port security inside a larger snooping/DAI story. This one is only ACL + port security.

**Time:** about 45–90 minutes.

---

# Key Concepts (Quick Review)

| Idea | Remember |
| ---- | -------- |
| **Port security** | L2 — MAC limit / sticky / violation on the **switch** |
| **ACL** | L3/L4 — permit/deny IP (and ports) on the **router** |
| **Sticky MAC** | First host’s MAC is learned, then treated like a static allow |
| **Violation shutdown** | Wrong MAC → port **err-disabled** until you recover it |
| **Extended ACL `in`** | Filters traffic **entering** that router interface |

**Mental model:** Port security answers “who is plugged into this jack?” ACL answers “what IP traffic may cross the router?”

---

# Network Topology

```text
                         R1 (2911)
                    g0/0 — router-on-a-stick
                    .10 = 192.168.10.1  (VLAN 10 STAFF)
                    .20 = 192.168.20.1  (VLAN 20 GUEST)
                    .30 = 192.168.30.1  (VLAN 30 SERVERS)
                              |
                         Trunk Fa0/24
                              |
                           SW1 (2960)
            +-----------------+------------------+
            |                 |                  |
         Fa0/1             Fa0/2              Fa0/3
         VLAN 10           VLAN 20            VLAN 30
       Port Security         |                  |
            |                 |                  |
          PC0               PC1              Server0
         (Staff)          (Guest)           (HTTP target)
```

Topology diagram: `CCNA_ACL_Port_Security_Topology.puml` (same folder).

---

# Devices Needed

| Device | Role |
| ------ | ---- |
| 1× Router (2911) – `R1` | Inter-VLAN routing + ACL |
| 1× Switch (2960) – `SW1` | VLANs + port security |
| 2× PC – `PC0`, `PC1` | Staff and Guest |
| 1× Server – `Server0` | Target for ACL tests |

**Cabling (Copper Straight-Through):**

| Link | Ports |
| ---- | ----- |
| R1 ↔ SW1 | G0/0 ↔ Fa0/24 |
| PC0 ↔ SW1 | Fa0 ↔ Fa0/1 |
| PC1 ↔ SW1 | Fa0 ↔ Fa0/2 |
| Server0 ↔ SW1 | Fa0 ↔ Fa0/3 |

---

# Address Plan

| VLAN | Name | Network | Gateway |
| ---- | ---- | ------- | ------- |
| 10 | STAFF | 192.168.10.0/24 | 192.168.10.1 |
| 20 | GUEST | 192.168.20.0/24 | 192.168.20.1 |
| 30 | SERVERS | 192.168.30.0/24 | 192.168.30.1 |

| Device | VLAN | IP | Gateway |
| ------ | ---- | -- | ------- |
| PC0 | 10 | 192.168.10.10 | 192.168.10.1 |
| PC1 | 20 | 192.168.20.10 | 192.168.20.1 |
| Server0 | 30 | 192.168.30.10 | 192.168.30.1 |

Use **static** IPs on all three hosts (no DHCP in this lab — keeps the focus on ACL and port security).

---

# Step 1: VLANs, Access Ports, Trunk (SW1)

```cisco
enable
configure terminal
hostname SW1

vlan 10
 name STAFF
vlan 20
 name GUEST
vlan 30
 name SERVERS

interface fa0/1
 switchport mode access
 switchport access vlan 10
 no shutdown

interface fa0/2
 switchport mode access
 switchport access vlan 20
 no shutdown

interface fa0/3
 switchport mode access
 switchport access vlan 30
 no shutdown

interface fa0/24
 switchport mode trunk
 no shutdown

end
write memory
```

Verify:

```cisco
show vlan brief
show interfaces fa0/24 switchport
```

**Expected:** Fa0/1 = VLAN 10, Fa0/2 = VLAN 20, Fa0/3 = VLAN 30. Fa0/24 **Administrative Mode: trunk**.

---

# Step 2: Router-on-a-Stick (R1)

```cisco
enable
configure terminal
hostname R1

interface g0/0
 no shutdown

interface g0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface g0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0

interface g0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0

end
write memory
```

On **SW1**, confirm the trunk is up:

```cisco
show interfaces trunk
```

---

# Step 3: Configure End Devices

### PC0 (Staff)

- IP: `192.168.10.10`
- Mask: `255.255.255.0`
- Gateway: `192.168.10.1`

### PC1 (Guest)

- IP: `192.168.20.10`
- Mask: `255.255.255.0`
- Gateway: `192.168.20.1`

### Server0

- IP: `192.168.30.10`
- Mask: `255.255.255.0`
- Gateway: `192.168.30.1`

Optional: on Server0 → **Services** → **HTTP** → On (so you can browse later as well as ping).

---

# Step 4: Baseline Connectivity (Before Security)

From **PC0**:

```text
ping 192.168.10.1
ping 192.168.30.10
ping 192.168.20.10
```

From **PC1**:

```text
ping 192.168.20.1
ping 192.168.30.10
```

**Expected:** All succeed. Fix VLANs / trunk / gateways before adding security.

---

# Step 5: Port Security on Fa0/1 (Staff Port)

Lock the Staff jack so only one MAC (PC0) is allowed.

On **SW1**:

```cisco
configure terminal

interface fa0/1
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown

end
```

Generate traffic from PC0 so the sticky MAC is learned:

```text
ping 192.168.10.1
```

Verify:

```cisco
show port-security interface fa0/1
show port-security address
```

**Expected — `show port-security interface fa0/1`:**

```text
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Shutdown
Maximum MAC Addresses      : 1
Total MAC Addresses        : 1
Sticky MAC Addresses       : 1
Security Violation Count   : 0
```

Save so sticky learning is kept:

```cisco
write memory
```

### Violation test

1. Unplug **PC0** from Fa0/1.
2. Plug **PC1** (or another PC) into Fa0/1.
3. From that device, `ping 192.168.10.1`.

**Expected:** Port goes **Secure-shutdown** / **err-disabled**.

```cisco
show port-security interface fa0/1
show interfaces status
```

**What to notice:** Port Status = **Secure-shutdown**; Fa0/1 = **err-disabled**.

### Recover

```cisco
configure terminal
interface fa0/1
 shutdown
 no shutdown
end
```

Reconnect **PC0** to Fa0/1. Confirm **Secure-up** again and that PC0 can ping the gateway.

**Optional softer mode** (count violations without killing the port):

```cisco
switchport port-security violation restrict
```

---

# Step 6: Extended ACL — Block Guest → Server

**Policy:**

| Source | Destination | Result |
| ------ | ----------- | ------ |
| Guest (192.168.20.0/24) | Server (192.168.30.0/24) | **Deny** |
| Everything else | — | **Permit** |

Apply the ACL **inbound** on the Guest subinterface so bad traffic is dropped as it enters the router.

On **R1**:

```cisco
configure terminal

access-list 101 deny ip 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255
access-list 101 permit ip any any

interface g0/0.20
 ip access-group 101 in

end
write memory
```

Verify:

```cisco
show access-lists
show ip interface g0/0.20
```

**Expected:** ACL 101 listed; `Incoming access list is 101` on `g0/0.20`.

---

# Step 7: ACL Tests

### From Staff (PC0) — should work

```text
ping 192.168.30.10
```

**Expected:** Success.

### From Guest (PC1) — should fail

```text
ping 192.168.30.10
```

**Expected:** Fail (timeout / unreachable).

### From Guest to Staff — should still work

```text
ping 192.168.10.10
```

**Expected:** Success (`permit ip any any` after the deny).

On R1, re-check hit counts:

```cisco
show access-lists 101
```

**What to notice:** The **deny** line increments when Guest pings the server; the **permit** line increments on allowed traffic.

---

# Step 8: End-to-End Checklist

| Check | Action / command | Pass if |
| ----- | ---------------- | ------- |
| VLANs | `show vlan brief` | Fa0/1=10, Fa0/2=20, Fa0/3=30 |
| Trunk | `show interfaces trunk` | Fa0/24 trunking |
| Port security | `show port-security interface fa0/1` | Enabled, Secure-up, max 1, sticky 1 |
| Violation | Wrong host on Fa0/1 | Secure-shutdown / err-disabled |
| ACL applied | `show ip interface g0/0.20` | Incoming ACL **101** |
| Staff → Server | PC0 `ping 192.168.30.10` | Success |
| Guest → Server | PC1 `ping 192.168.30.10` | Fail |
| Guest → Staff | PC1 `ping 192.168.10.10` | Success |

---

# Troubleshooting

| Symptom | Likely cause | Fix |
| ------- | ------------ | --- |
| No cross-VLAN pings before ACL | Trunk / subinterface / gateway | Check `show interfaces trunk`, `show ip int brief` on R1 |
| Port never Secure-up | No traffic from PC0 yet | Ping gateway from PC0, then re-check |
| Sticky MAC wrong after swap | Old sticky still learned | Clear sticky / recover port; reconnect PC0 |
| Guest still reaches Server | ACL not applied or wrong direction | Confirm `ip access-group 101 in` on **g0/0.20** |
| Staff blocked too | Deny too broad / wrong ACL | Deny only `192.168.20.0` → `192.168.30.0`; keep `permit ip any any` |

---

# Skills Learned

- Configure sticky port security with max 1 and violation shutdown
- Prove and recover from a port-security violation
- Write an extended ACL with a specific deny + permit any
- Apply an ACL inbound on a router-on-a-stick subinterface
- Tell port security (MAC / L2) apart from ACLs (IP / L3)

**Maps to CCNA domains:** Network Access (port security), Security Fundamentals (ACLs).

---

# Common Mistakes

| Mistake | Fix |
| ------- | --- |
| Thinking port security filters by IP | It does not — use an ACL for IP policy |
| Applying ACL on the wrong subinterface | Guest source → apply on **g0/0.20 in** |
| Forgetting `permit ip any any` | Implicit deny would kill all other Guest traffic |
| Enabling port security before learning sticky | Generate traffic from the legit PC first |
| Leaving wrong host on Fa0/1 after demo | Recover port, reconnect PC0, re-verify Secure-up |

---

# What to Save

- Packet Tracer file: `ACL_Port_Security.pkt` (optional, in this folder)
- Output: `show port-security interface fa0/1` (before and after violation)
- Output: `show access-lists 101` after Guest and Staff tests
- One-line note: “Port security = who on the jack; ACL = what IP traffic may route”

---

# Reflection Prompts

1. Why does Guest → Server get filtered on the **router**, while a swapped laptop on Fa0/1 is stopped on the **switch**?
2. What would change if you put ACL 101 **out** on `g0/0.30` instead of **in** on `g0/0.20`?
3. If you set port-security maximum to **2**, what attack or misuse still gets through that an ACL would not catch?
