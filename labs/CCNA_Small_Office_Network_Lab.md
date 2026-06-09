# CCNA Packet Tracer Lab – Small Office Network (Simplified)

**Do steps in order.** After each CLI step, run **`write memory`** on that device. Save your `.pkt` when done.

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

| VLAN | Name     | Network         | Gateway      |
| ---- | -------- | --------------- | ------------ |
| 10   | HR       | 192.168.10.0/24 | 192.168.10.1 |
| 20   | Sales    | 192.168.20.0/24 | 192.168.20.1 |
| 30   | Servers  | 192.168.30.0/24 | 192.168.30.1 |
| 40   | Printers | 192.168.40.0/24 | 192.168.40.1 |

| Device   | VLAN | IP            | Gateway      |
| -------- | ---- | ------------- | ------------ |
| PC0      | 10   | DHCP (or .10) | 192.168.10.1 |
| PC1      | 10   | DHCP (or .11) | 192.168.10.1 |
| PC2      | 20   | DHCP (or .10) | 192.168.20.1 |
| PC3      | 20   | DHCP (or .11) | 192.168.20.1 |
| Server0  | 30   | 192.168.30.10 | 192.168.30.1 |
| Printer0 | 40   | 192.168.40.10 | 192.168.40.1 |

---

## Step 0 — Build and Cable

**Devices:** 1 router (2911), 2 switches (2960), 4 PCs, 1 server, 1 printer.

**Cable everything with Copper Straight-Through:**

| From   | Port   | To      | Port   |
| ------ | ------ | ------- | ------ |
| R1     | Gig0/0 | SW1     | Fa0/24 |
| SW1    | Fa0/1  | PC0     | Fa0    |
| SW1    | Fa0/2  | PC1     | Fa0    |
| SW1    | Fa0/23 | SW2     | Fa0/23 |
| SW2    | Fa0/1  | PC2     | Fa0    |
| SW2    | Fa0/2  | PC3     | Fa0    |
| SW2    | Fa0/3  | Server0 | Fa0    |
| SW2    | Fa0/4  | Printer0| Fa0    |

```text
        R1
        |
       SW1
      / | \
   PC0 PC1 SW2
            / | | \
         PC2 PC3 Server0
                    \
                  Printer0
```

**Full diagram (VLANs, interfaces, router-on-a-stick, legend):**

- Source: [CCNA_Small_Office_Network_Topology.puml](CCNA_Small_Office_Network_Topology.puml)
- Image: [CCNA_Small_Office_Network_Topology.png](CCNA_Small_Office_Network_Topology.png)

Render PNG: `plantuml labs/CCNA_Small_Office_Network_Topology.puml` or paste the `.puml` file into [PlantUML online](https://www.plantuml.com/plantuml).

---

## Step 1 — Create VLANs (SW1 and SW2)

**Run the same commands on BOTH switches.**

```cisco
enable
configure terminal
vlan 10
 name HR
exit
vlan 20
 name SALES
exit
vlan 30
 name SERVERS
exit
vlan 40
 name PRINTERS
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
10   HR                               active
20   SALES                            active
30   SERVERS                          active
40   PRINTERS                         active
```

Port numbers under VLAN 1 will differ until Steps 2–4 — that is OK. **Look for VLANs 10, 20, 30, 40** with correct names.

Run on **SW1**, then repeat on **SW2** (including `write memory` on each).

---

## Step 2 — Assign PCs to VLANs

### SW1

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

### SW2

```cisco
enable
configure terminal
interface range fa0/1-2
 switchport mode access
 switchport access vlan 20
exit
interface fa0/3
 switchport mode access
 switchport access vlan 30
exit
interface fa0/4
 switchport mode access
 switchport access vlan 40
exit
end
write memory
```

**Check:** `show vlan brief` — PCs on correct VLANs.

**Expected output — SW1:**

```text
Switch# show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/3-22, Fa0/23-24, ...
10   HR                               active    Fa0/1, Fa0/2
20   SALES                            active
30   SERVERS                          active
40   PRINTERS                         active
```

**Expected output — SW2:**

```text
Switch# show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/5-22, Fa0/23-24, ...
10   HR                               active
20   SALES                            active    Fa0/1, Fa0/2
30   SERVERS                          active    Fa0/3
40   PRINTERS                         active    Fa0/4
```

**Look for:** Fa0/1–2 on SW1 under VLAN **10**; Fa0/1–2 on SW2 under VLAN **20**; Fa0/3 = VLAN **30**; Fa0/4 = VLAN **40**.

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

## Step 5 — Router-on-a-Stick (R1 only)

Paste **all** of this on the router:

```cisco
enable
configure terminal
hostname R1
interface g0/0
 no shutdown
exit
interface g0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
exit
interface g0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
exit
interface g0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
exit
interface g0/0.40
 encapsulation dot1Q 40
 ip address 192.168.40.1 255.255.255.0
exit
end
write memory
```

**Check:** `show ip interface brief`

**Expected output:**

```text
GigabitEthernet0/0     unassigned      YES unset  up                    up
GigabitEthernet0/0.10  192.168.10.1    YES manual up                    up
GigabitEthernet0/0.20  192.168.20.1    YES manual up                    up
GigabitEthernet0/0.30  192.168.30.1    YES manual up                    up
GigabitEthernet0/0.40  192.168.40.1    YES manual up                    up
```

- Parent `Gig0/0` has **no IP** — that is normal.
- You need **four subinterfaces** with IPs ending in `.1`.

**Test:** Set PC0 to `192.168.10.10` / gateway `192.168.10.1`. Set PC2 to `192.168.20.10` / gateway `192.168.20.1`. From PC0: `ping 192.168.20.10`

**Expected ping (PC0 Command Prompt):**

```text
Ping 192.168.20.10 with 32 bytes of data:

Reply from 192.168.20.10: bytes=32 time=1ms TTL=127
Reply from 192.168.20.10: bytes=32 time=1ms TTL=127
Reply from 192.168.20.10: bytes=32 time=1ms TTL=127
Reply from 192.168.20.10: bytes=32 time=1ms TTL=127

Ping statistics for 192.168.20.10:
    Packets: Sent = 4, Received = 4, Lost = 0
```

TTL may differ — **4 replies = inter-VLAN routing works**.

---

## Step 6 — Server and Printer (GUI)

Click device → **Desktop** → **IP Configuration** → **Static**:

| Device   | IP            | Mask          | Gateway      |
| -------- | ------------- | ------------- | ------------ |
| Server0  | 192.168.30.10 | 255.255.255.0 | 192.168.30.1 |
| Printer0 | 192.168.40.10 | 255.255.255.0 | 192.168.40.1 |

**Test from PC0:** `ping 192.168.30.10`

**Expected:**

```text
Reply from 192.168.30.10: bytes=32 time=1ms TTL=127
...
Packets: Sent = 4, Received = 4, Lost = 0
```

---

## Step 7 — DNS on Server (GUI)

1. **Server0** → **Services** → **DNS** → **On**
2. Add A record: `company.local` → `192.168.30.10`

**Test from PC0:** `ping company.local` (set DNS to `192.168.30.10` on PC if needed)

**Expected:**

```text
Pinging 192.168.30.10 with 32 bytes of data:

Reply from 192.168.30.10: bytes=32 time=1ms TTL=127
...
```

Name resolves to **192.168.30.10** — DNS works.

---

## Step 8 — HTTP on Server (GUI)

1. **Server0** → **Services** → **HTTP** → **On**
2. On PC0 → **Web Browser** → `http://company.local`

**Expected:** Browser shows the server page (e.g. "Welcome to Company Network" if you edited it).

---

## Step 9 — DHCP on Router (R1)

```cisco
enable
configure terminal
ip dhcp excluded-address 192.168.10.10 192.168.10.11
ip dhcp excluded-address 192.168.20.10 192.168.20.11
ip dhcp excluded-address 192.168.30.10
ip dhcp excluded-address 192.168.40.10
ip dhcp pool HR
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 192.168.30.10
exit
ip dhcp pool SALES
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 192.168.30.10
exit
ip dhcp pool SERVERS
 network 192.168.30.0 255.255.255.0
 default-router 192.168.30.1
 dns-server 192.168.30.10
exit
ip dhcp pool PRINTERS
 network 192.168.40.0 255.255.255.0
 default-router 192.168.40.1
 dns-server 192.168.30.10
exit
end
write memory
```

**On all 4 PCs:** Desktop → IP Configuration → **DHCP**

**Check:** `show ip dhcp binding` on R1; `ipconfig` on PCs.

**Expected — R1:**

```text
R1# show ip dhcp binding

IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.10.x     0100.xxxx.xxxx.xx       --                      Automatic
192.168.20.x     0100.xxxx.xxxx.xx       --                      Automatic
```

Exact IPs and MACs vary. You should see leases in **192.168.10.x** and **192.168.20.x** after PCs use DHCP.

**Expected — PC0 (`ipconfig` in Command Prompt):**

```text
IP Address...............: 192.168.10.x
Subnet Mask..............: 255.255.255.0
Default Gateway..........: 192.168.10.1
DNS Servers..............: 192.168.30.10
```

**Expected — PC2:** IP in **192.168.20.x**, gateway **192.168.20.1**, DNS **192.168.30.10**.

---

## Step 10 — ACL: Block Sales from HR (R1)

```cisco
enable
configure terminal
access-list 100 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
access-list 100 permit ip any any
interface g0/0.20
 ip access-group 100 in
end
write memory
```

**Test:**

From **PC2** (Sales): `ping 192.168.10.10` — **fail**

**Expected (PC2):**

```text
Request timed out.
Request timed out.
Request timed out.
Request timed out.

Ping statistics for 192.168.10.10:
    Packets: Sent = 4, Received = 0, Lost = 4
```

From **PC0** (HR): `ping 192.168.20.10` — **work**

**Expected (PC0):**

```text
Reply from 192.168.20.10: bytes=32 time=1ms TTL=127
...
Packets: Sent = 4, Received = 4, Lost = 0
```

**Verify ACL on R1:**

```text
R1# show access-lists

Extended IP access list 100
    deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
    permit ip any any (xx matches)
```

`xx matches` increases when Sales tries to reach HR — ACL is working.

---

## Step 11 — Port Security (SW1, port to PC0)

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

**Check:** `show port-security interface fa0/1`

**Expected output (SW1):**

```text
Switch# show port-security interface fa0/1
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Shutdown
Maximum MAC Addresses    : 1
Total MAC Addresses        : 1
Configured MAC Addresses   : 1
Sticky MAC Addresses       : 1
Last Source Address        : xxxx.xxxx.xxxx
Security Violation Count   : 0
```

MAC address will match PC0. **Enabled** and **Secure-up** = good.

---

## Step 12 — Spanning Tree (optional read)

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

## Step 13 — Save Lab File

**Packet Tracer:** **File → Save As** your `.pkt` file.

(Optional final save on all three devices if you skipped `write memory` earlier: `enable` → `write memory`.)

---

## Done Checklist

| Test | Command | Should work? |
| ---- | ------- | ------------ |
| Same VLAN | PC0: `ping 192.168.10.11` | Yes — 4 replies |
| Cross VLAN | PC0: `ping 192.168.20.10` | Yes — 4 replies |
| Server | PC0: `ping 192.168.30.10` | Yes — 4 replies |
| DNS | PC0: `ping company.local` | Yes — resolves to .30.10 |
| ACL block | PC2: `ping 192.168.10.10` | No — 0 replies, timed out |
| Trunks | SW1: `show interfaces trunk` | Fa0/23, Fa0/24 trunking |
| Router | R1: `show ip interface brief` | Gig0/0.10–.40 up/up with .1 IPs |
| DHCP | R1: `show ip dhcp binding` | Leases in 10.x and 20.x |
| Port security | SW1: `show port-security interface fa0/1` | Enabled, Secure-up |

---

## Quick Fixes

| Problem | Fix |
| ------- | --- |
| `Invalid input` on `interface` | Run `enable` then `configure terminal` first |
| No subinterfaces on router | Complete Step 5; run `write memory` |
| `Gig0/0` administratively down | `interface g0/0` → `no shutdown` |
| No IPs on router | IPs go on **subinterfaces** (g0/0.10), not Gig0/0 |
| VLANs can't talk | Check Steps 3, 4, 5 (trunks + subinterfaces) |
| DHCP fails | Check Step 9 pools and PC set to DHCP |
