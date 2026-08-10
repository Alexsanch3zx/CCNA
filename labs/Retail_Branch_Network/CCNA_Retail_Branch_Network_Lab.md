# CCNA Packet Tracer Lab – Retail Branch Network

**Do steps in order.** After each CLI step, run **`write memory`** on that device. Save your `.pkt` when done.

This lab is the same **style** as the [Small Office Network Lab](../Small_Office_Network/CCNA_Small_Office_Network_Lab.md) (VLANs, trunks, router-on-a-stick, DHCP, DNS/HTTP, ACL, port security), with a **retail branch** story plus two new skills:

- **Guest VLAN isolation** — guests cannot reach Staff, POS, or Servers
- **PAT to an ISP stub** — Staff/POS/Guest reach a simulated Internet via overload NAT

**Related notes:** [NAT / PAT](../../IP-addresses/NAT_PAT_Notes.md) · [ACL Study Guide](../../ACL-Study-Guide.md)

---

## Before Every CLI Step

You must be in config mode. If you see `Switch>` or `Router>`:

```cisco
enable
configure terminal
```

Paste the commands for that step. When the block ends with `end`, run:

```cisco
write memory
```

That saves the config so it survives a reload.

---

## Quick Reference

| VLAN | Name    | Network        | Gateway     |
| ---- | ------- | -------------- | ----------- |
| 10   | Staff   | 10.10.10.0/24  | 10.10.10.1  |
| 20   | POS     | 10.10.20.0/24  | 10.10.20.1  |
| 30   | Servers | 10.10.30.0/24  | 10.10.30.1  |
| 40   | Guest   | 10.10.40.0/24  | 10.10.40.1  |

| Segment  | Network           | Notes                          |
| -------- | ----------------- | ------------------------------ |
| R1 ↔ R2  | 203.0.113.0/30    | R1 `.1`, R2 `.2` (WAN)         |
| ISP LAN  | 203.0.113.8/29    | R2 `.9`, Server1 `.10`         |

| Device   | VLAN | IP             | Gateway      |
| -------- | ---- | -------------- | ------------ |
| PC0      | 10   | DHCP (or .10)  | 10.10.10.1   |
| PC1      | 10   | DHCP (or .11)  | 10.10.10.1   |
| PC2      | 20   | DHCP (or .10)  | 10.10.20.1   |
| PC3      | 40   | DHCP (or .10)  | 10.10.40.1   |
| Server0  | 30   | 10.10.30.10    | 10.10.30.1   |
| Server1  | ISP  | 203.0.113.10   | 203.0.113.9  |

---

## Step 0 — Build and Cable

**Devices:** 2 routers (2911) — `R1` (branch) and `R2` (ISP); 2 switches (2960); 4 PCs; 2 servers.

**Cable everything with Copper Straight-Through** (use Copper Cross-Over only if your PT version requires it for router↔router):

| From    | Port   | To      | Port   |
| ------- | ------ | ------- | ------ |
| R1      | Gig0/0 | SW1     | Fa0/24 |
| R1      | Gig0/1 | R2      | Gig0/0 |
| R2      | Gig0/1 | Server1 | Fa0    |
| SW1     | Fa0/1  | PC0     | Fa0    |
| SW1     | Fa0/2  | PC1     | Fa0    |
| SW1     | Fa0/23 | SW2     | Fa0/23 |
| SW2     | Fa0/1  | PC2     | Fa0    |
| SW2     | Fa0/2  | PC3     | Fa0    |
| SW2     | Fa0/3  | Server0 | Fa0    |

```text
  Server1 (ISP HTTP)
        |
       R2 ---- WAN ---- R1
                         |
                        SW1
                       / | \
                    PC0 PC1 SW2
                           / | \
                        PC2 PC3 Server0
                       (POS)(Guest)(store)
```

**Full diagram (VLANs, WAN, PAT, Guest ACL, legend):**

- Source: [CCNA_Retail_Branch_Network_Topology.puml](CCNA_Retail_Branch_Network_Topology.puml)
- Image: [CCNA_Retail_Branch_Network_Topology.png](CCNA_Retail_Branch_Network_Topology.png)

Render PNG: `plantuml labs/Retail_Branch_Network/CCNA_Retail_Branch_Network_Topology.puml` or paste the `.puml` into [PlantUML online](https://www.plantuml.com/plantuml).

---

## Step 1 — Create VLANs (SW1 and SW2)

**Run the same commands on BOTH switches.**

```cisco
enable
configure terminal
vlan 10
 name STAFF
exit
vlan 20
 name POS
exit
vlan 30
 name SERVERS
exit
vlan 40
 name GUEST
exit
end
write memory
```

**Check:** `show vlan brief` — VLANs 10, 20, 30, 40 listed.

**Expected output (SW1 or SW2 — same VLAN list):**

```text
Switch# show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/3-22, Fa0/24-48, Gig0/1-2
10   STAFF                            active
20   POS                              active
30   SERVERS                          active
40   GUEST                            active
```

Port numbers under VLAN 1 will differ until later steps — that is OK. **Look for VLANs 10, 20, 30, 40** with correct names.

Run on **SW1**, then repeat on **SW2** (including `write memory` on each).

---

## Step 2 — Assign End Devices to VLANs

### SW1 (Staff PCs)

```cisco
enable
configure terminal
interface range fa0/1-2
 switchport mode access
 switchport access vlan 10
exit
end
write memory
```

### SW2 (POS, Guest, Server)

```cisco
enable
configure terminal
interface fa0/1
 switchport mode access
 switchport access vlan 20
exit
interface fa0/2
 switchport mode access
 switchport access vlan 40
exit
interface fa0/3
 switchport mode access
 switchport access vlan 30
exit
end
write memory
```

**Check:** `show vlan brief` — ports on correct VLANs.

**Expected output — SW1:**

```text
Switch# show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/3-22, Fa0/23-24, ...
10   STAFF                            active    Fa0/1, Fa0/2
20   POS                              active
30   SERVERS                          active
40   GUEST                            active
```

**Expected output — SW2:**

```text
Switch# show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/4-22, Fa0/23-24, ...
10   STAFF                            active
20   POS                              active    Fa0/1
30   SERVERS                          active    Fa0/3
40   GUEST                            active    Fa0/2
```

**Look for:** SW1 Fa0/1–2 under VLAN **10**; SW2 Fa0/1 = **20**, Fa0/2 = **40**, Fa0/3 = **30**.

---

## Step 3 — Trunk Between Switches (SW1 and SW2)

**On BOTH switches:**

```cisco
enable
configure terminal
interface fa0/23
 switchport mode trunk
exit
end
write memory
```

**Check:** `show interfaces trunk` — Fa0/23 is trunking.

**Expected output (after Step 3, on SW1 or SW2):**

```text
Switch# show interfaces trunk

Port        Mode         Encapsulation  Status        Native vlan
Fa0/23      on           802.1q         trunking      1

Port        Vlans allowed on trunk
Fa0/23      1-4094
```

If **empty** — cable missing, port down, or trunk not saved. Run `write memory` and check link to other switch.

Run on **SW1**, then **SW2** (including `write memory` on each).

---

## Step 4 — Trunk to Router (SW1 only)

```cisco
enable
configure terminal
interface fa0/24
 switchport mode trunk
exit
end
write memory
```

**Check:** `show interfaces trunk` — Fa0/23 **and** Fa0/24 are trunks.

**Expected output (SW1 after Step 4 + R1 Gig0/0 up):**

```text
Switch# show interfaces trunk

Port        Mode         Encapsulation  Status        Native vlan
Fa0/23      on           802.1q         trunking      1
Fa0/24      on           802.1q         trunking      1

Port        Vlans allowed on trunk
Fa0/23      1-4094
Fa0/24      1-4094
```

**Also check link is up:**

```text
Switch# show interfaces fa0/24
...
FastEthernet0/24 is up, line protocol is up
```

If Fa0/24 missing from trunk table — run Step 4 again. If **down/down** — cable R1 Gig0/0 and run `no shutdown` on R1 `g0/0` (Step 5).

---

## Step 5 — Router-on-a-Stick (R1 LAN side)

Paste **all** of this on **R1**:

```cisco
enable
configure terminal
hostname R1
interface g0/0
 no shutdown
exit
interface g0/0.10
 encapsulation dot1Q 10
 ip address 10.10.10.1 255.255.255.0
exit
interface g0/0.20
 encapsulation dot1Q 20
 ip address 10.10.20.1 255.255.255.0
exit
interface g0/0.30
 encapsulation dot1Q 30
 ip address 10.10.30.1 255.255.255.0
exit
interface g0/0.40
 encapsulation dot1Q 40
 ip address 10.10.40.1 255.255.255.0
exit
end
write memory
```

**Check:** `show ip interface brief`

**Expected output:**

```text
GigabitEthernet0/0     unassigned      YES unset  up                    up
GigabitEthernet0/0.10  10.10.10.1      YES manual up                    up
GigabitEthernet0/0.20  10.10.20.1      YES manual up                    up
GigabitEthernet0/0.30  10.10.30.1      YES manual up                    up
GigabitEthernet0/0.40  10.10.40.1      YES manual up                    up
```

- Parent `Gig0/0` has **no IP** — that is normal.
- You need **four subinterfaces** with IPs ending in `.1`.

**Test:** Set PC0 to `10.10.10.10` / gateway `10.10.10.1`. Set PC2 to `10.10.20.10` / gateway `10.10.20.1`. From PC0: `ping 10.10.20.10`

**Expected ping (PC0 Command Prompt):**

```text
Ping 10.10.20.10 with 32 bytes of data:

Reply from 10.10.20.10: bytes=32 time=1ms TTL=127
Reply from 10.10.20.10: bytes=32 time=1ms TTL=127
Reply from 10.10.20.10: bytes=32 time=1ms TTL=127
Reply from 10.10.20.10: bytes=32 time=1ms TTL=127

Ping statistics for 10.10.20.10:
    Packets: Sent = 4, Received = 4, Lost = 0
```

TTL may differ — **4 replies = inter-VLAN routing works**.

---

## Step 6 — Internal Server (GUI)

Click **Server0** → **Desktop** → **IP Configuration** → **Static**:

| Device  | IP          | Mask          | Gateway     |
| ------- | ----------- | ------------- | ----------- |
| Server0 | 10.10.30.10 | 255.255.255.0 | 10.10.30.1  |

**Test from PC0:** `ping 10.10.30.10`

**Expected:**

```text
Reply from 10.10.30.10: bytes=32 time=1ms TTL=127
...
Packets: Sent = 4, Received = 4, Lost = 0
```

---

## Step 7 — DNS on Server0 (GUI)

1. **Server0** → **Services** → **DNS** → **On**
2. Add A record: `store.local` → `10.10.30.10`

**Test from PC0:** set DNS to `10.10.30.10`, then `ping store.local`

**Expected:**

```text
Pinging 10.10.30.10 with 32 bytes of data:

Reply from 10.10.30.10: bytes=32 time=1ms TTL=127
...
```

Name resolves to **10.10.30.10** — DNS works.

---

## Step 8 — HTTP on Server0 (GUI)

1. **Server0** → **Services** → **HTTP** → **On**
2. On PC0 → **Web Browser** → `http://store.local`

**Expected:** Browser shows the server page (edit the HTML if you want a “Welcome to Retail Branch” banner).

---

## Step 9 — DHCP on Router (R1)

```cisco
enable
configure terminal
ip dhcp excluded-address 10.10.10.1 10.10.10.11
ip dhcp excluded-address 10.10.20.1 10.10.20.10
ip dhcp excluded-address 10.10.30.1 10.10.30.10
ip dhcp excluded-address 10.10.40.1 10.10.40.10
ip dhcp pool STAFF
 network 10.10.10.0 255.255.255.0
 default-router 10.10.10.1
 dns-server 10.10.30.10
exit
ip dhcp pool POS
 network 10.10.20.0 255.255.255.0
 default-router 10.10.20.1
 dns-server 10.10.30.10
exit
ip dhcp pool GUEST
 network 10.10.40.0 255.255.255.0
 default-router 10.10.40.1
 dns-server 10.10.30.10
exit
end
write memory
```

**On PC0, PC1, PC2, PC3:** Desktop → IP Configuration → **DHCP**

**Check:** `show ip dhcp binding` on R1; `ipconfig` on PCs.

**Expected — R1:**

```text
R1# show ip dhcp binding

IP address       Client-ID/              Lease expiration        Type
                 Hardware address
10.10.10.x       0100.xxxx.xxxx.xx       --                      Automatic
10.10.20.x       0100.xxxx.xxxx.xx       --                      Automatic
10.10.40.x       0100.xxxx.xxxx.xx       --                      Automatic
```

Exact IPs and MACs vary. You should see leases in **10.10.10.x**, **10.10.20.x**, and **10.10.40.x** after PCs use DHCP.

**Expected — PC0 (`ipconfig`):**

```text
IP Address...............: 10.10.10.x
Subnet Mask..............: 255.255.255.0
Default Gateway..........: 10.10.10.1
DNS Servers..............: 10.10.30.10
```

**Expected — PC2 (POS):** IP in **10.10.20.x**, gateway **10.10.20.1**.  
**Expected — PC3 (Guest):** IP in **10.10.40.x**, gateway **10.10.40.1**.

---

## Step 10 — ISP Stub (R2 + Server1)

### R2 (ISP router)

```cisco
enable
configure terminal
hostname R2
interface g0/0
 description WAN-to-R1
 ip address 203.0.113.2 255.255.255.252
 no shutdown
exit
interface g0/1
 description ISP-LAN
 ip address 203.0.113.9 255.255.255.248
 no shutdown
exit
end
write memory
```

### R1 WAN interface (toward ISP)

```cisco
enable
configure terminal
interface g0/1
 description WAN-to-ISP
 ip address 203.0.113.1 255.255.255.252
 no shutdown
exit
end
write memory
```

**Check on R1:** `ping 203.0.113.2` — should succeed.

**Expected:**

```text
R1# ping 203.0.113.2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 203.0.113.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5)
```

### Server1 (ISP web target) — GUI

| Device  | IP           | Mask            | Gateway      |
| ------- | ------------ | --------------- | ------------ |
| Server1 | 203.0.113.10 | 255.255.255.248 | 203.0.113.9  |

1. **Server1** → **Services** → **HTTP** → **On**
2. Optional DNS A record on Server1: `isp.example` → `203.0.113.10`

**Check from R2:** `ping 203.0.113.10`

---

## Step 11 — Default Route + PAT on R1

Staff PCs use private `10.10.x.x` addresses. The ISP only knows the public WAN. **PAT (overload)** lets many inside hosts share R1’s WAN IP. See [NAT / PAT notes](../../IP-addresses/NAT_PAT_Notes.md).

```cisco
enable
configure terminal
ip route 0.0.0.0 0.0.0.0 203.0.113.2
access-list 1 permit 10.10.0.0 0.0.255.255
ip nat inside source list 1 interface GigabitEthernet0/1 overload
interface g0/0.10
 ip nat inside
exit
interface g0/0.20
 ip nat inside
exit
interface g0/0.30
 ip nat inside
exit
interface g0/0.40
 ip nat inside
exit
interface g0/1
 ip nat outside
exit
end
write memory
```

**Why ACL 1?** Only traffic **sourced** from `10.10.0.0/16` is translated. That matches all branch VLANs.

**Test from PC0 (Staff):**

1. `ping 203.0.113.10` — should succeed after PAT
2. Web Browser → `http://203.0.113.10`

**Expected ping:**

```text
Reply from 203.0.113.10: bytes=32 time=…ms TTL=…
...
Packets: Sent = 4, Received = 4, Lost = 0
```

**Verify NAT on R1:**

```cisco
show ip nat translations
show ip route
```

**Expected shape:**

```text
R1# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 203.0.113.1:…     10.10.10.x:…       203.0.113.10:…     203.0.113.10:…

R1# show ip route
...
S*   0.0.0.0/0 [1/0] via 203.0.113.2
```

If ping fails: confirm `ip nat inside` on all four subinterfaces, `ip nat outside` on `g0/1`, and the default route.

---

## Step 12 — ACL: Block Guest from Internal Networks (R1)

Guest may use the Internet (via PAT) but must **not** reach Staff, POS, or Servers. Apply the ACL **inbound** on the Guest subinterface so traffic from Guest is filtered as it enters R1.

```cisco
enable
configure terminal
ip access-list extended GUEST-FILTER
 deny ip 10.10.40.0 0.0.0.255 10.10.10.0 0.0.0.255
 deny ip 10.10.40.0 0.0.0.255 10.10.20.0 0.0.0.255
 deny ip 10.10.40.0 0.0.0.255 10.10.30.0 0.0.0.255
 permit ip any any
exit
interface g0/0.40
 ip access-group GUEST-FILTER in
exit
end
write memory
```

**Test:**

| From | To | Result |
| ---- | -- | ------ |
| PC3 (Guest) | `ping 10.10.10.10` (Staff) | **Fail** — timed out |
| PC3 (Guest) | `ping 10.10.20.10` (POS) | **Fail** |
| PC3 (Guest) | `ping 10.10.30.10` (Server0) | **Fail** |
| PC3 (Guest) | `ping 203.0.113.10` (ISP) | **Work** — PAT still allowed |
| PC0 (Staff) | `ping 10.10.30.10` | **Work** — ACL only on Guest |

**Expected (PC3 → Staff):**

```text
Request timed out.
Request timed out.
Request timed out.
Request timed out.

Ping statistics for 10.10.10.10:
    Packets: Sent = 4, Received = 0, Lost = 4
```

**Expected (PC3 → ISP):**

```text
Reply from 203.0.113.10: bytes=32 time=…ms TTL=…
...
Packets: Sent = 4, Received = 4, Lost = 0
```

**Verify ACL on R1:**

```text
R1# show access-lists GUEST-FILTER

Extended IP access list GUEST-FILTER
    deny ip 10.10.40.0 0.0.0.255 10.10.10.0 0.0.0.255 (xx matches)
    deny ip 10.10.40.0 0.0.0.255 10.10.20.0 0.0.0.255
    deny ip 10.10.40.0 0.0.0.255 10.10.30.0 0.0.0.255
    permit ip any any (yy matches)
```

`xx matches` increases when Guest tries internal pings; `yy matches` increases on Guest → Internet traffic.

> **DNS note:** Guest DHCP still points at `10.10.30.10`. After this ACL, Guest DNS to Server0 is blocked. For Internet tests, use the **IP** `203.0.113.10` in the browser, or temporarily set Guest DNS to `203.0.113.10` if you added a record on Server1.

---

## Step 13 — Port Security (SW1, port to PC0)

```cisco
enable
configure terminal
interface fa0/1
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
exit
end
write memory
```

Generate traffic from PC0 first (`ping 10.10.10.1`) so the sticky MAC learns.

**Check:** `show port-security interface fa0/1`

**Expected output (SW1):**

```text
Switch# show port-security interface fa0/1
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Shutdown
Maximum MAC Addresses      : 1
Total MAC Addresses        : 1
Configured MAC Addresses   : 1
Sticky MAC Addresses       : 1
Last Source Address        : xxxx.xxxx.xxxx
Security Violation Count   : 0
```

MAC address will match PC0. **Enabled** and **Secure-up** = good.

**Optional demo:** Unplug PC0, plug another PC into Fa0/1, ping — port should go **Secure-shutdown** / err-disabled. Recover with `shutdown` then `no shutdown` on Fa0/1.

---

## Step 14 — Spanning Tree (optional read)

On SW1 or SW2:

```cisco
show spanning-tree
```

**Expected (sample — root may be SW1 or SW2):**

```text
Switch# show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     xxxx.xxxx.xxxx
             This bridge is the root
  ...

VLAN0010
  ...
```

Most ports show **FWD (Forwarding)** in this lab (no loop). You are learning to **read** the output, not change STP.

---

## Step 15 — Save Lab File

**Packet Tracer:** **File → Save As** your `.pkt` file.

(Optional final save on all devices if you skipped `write memory` earlier: `enable` → `write memory`.)

---

## Done Checklist

| Test | Command / action | Should work? |
| ---- | ---------------- | ------------ |
| Same VLAN | PC0: `ping` PC1 | Yes — 4 replies |
| Cross VLAN | PC0: `ping` PC2 (POS) | Yes — 4 replies |
| Server | PC0: `ping 10.10.30.10` | Yes |
| DNS | PC0: `ping store.local` | Yes — resolves to .30.10 |
| Internet (PAT) | PC0: `ping 203.0.113.10` | Yes |
| Guest blocked | PC3: `ping 10.10.30.10` | No — timed out |
| Guest Internet | PC3: `ping 203.0.113.10` | Yes |
| Trunks | SW1: `show interfaces trunk` | Fa0/23, Fa0/24 trunking |
| Router LAN | R1: `show ip interface brief` | Gig0/0.10–.40 up/up with .1 IPs |
| NAT | R1: `show ip nat translations` | Entries after Guest/Staff ping ISP |
| DHCP | R1: `show ip dhcp binding` | Leases in 10.x, 20.x, 40.x |
| Port security | SW1: `show port-security interface fa0/1` | Enabled, Secure-up |

---

## Quick Fixes

| Problem | Fix |
| ------- | --- |
| `Invalid input` on `interface` | Run `enable` then `configure terminal` first |
| No subinterfaces on router | Complete Step 5; run `write memory` |
| `Gig0/0` administratively down | `interface g0/0` → `no shutdown` |
| No IPs on router LAN | IPs go on **subinterfaces** (`g0/0.10`), not Gig0/0 |
| VLANs can't talk | Check Steps 3, 4, 5 (trunks + subinterfaces) |
| DHCP fails | Check Step 9 pools and PC set to DHCP |
| No Internet from LAN | Check Step 11: default route, `ip nat inside`/`outside`, ACL 1 |
| NAT translations empty | Generate traffic (`ping` ISP) then `show ip nat translations` |
| Guest still reaches Server0 | ACL on `g0/0.40 in`; confirm PC3 is VLAN 40 |
| Guest Internet broken after ACL | Last ACE must be `permit ip any any`; use ISP IP not `store.local` |
| R1 cannot ping R2 | Cable Gig0/1↔Gig0/0; matching `/30` masks; both `no shutdown` |

---

## Skills Learned

- Multi-VLAN retail design (Staff / POS / Servers / Guest)
- Switch trunks and access ports across two switches
- Router-on-a-stick inter-VLAN routing
- DHCP pools with exclusions and DNS option
- Internal DNS + HTTP services
- Default route toward an ISP stub
- **PAT (overload)** with inside/outside interfaces
- Extended ACL for Guest isolation while allowing Internet
- Port security (sticky MAC, violation shutdown)

## How This Complements the Small Office Lab

| Small Office | This lab |
| ------------ | -------- |
| HR / Sales / Servers / Printers | Staff / POS / Servers / Guest |
| ACL: Sales cannot reach HR | ACL: Guest cannot reach any internal VLAN |
| No WAN | ISP stub + default route |
| No NAT | PAT so private LANs reach “Internet” |
