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

---

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

**Test:** Set PC0 to `192.168.10.10` / gateway `192.168.10.1`. Set PC2 to `192.168.20.10` / gateway `192.168.20.1`. From PC0: `ping 192.168.20.10` — should work.

---

## Step 6 — Server and Printer (GUI)

Click device → **Desktop** → **IP Configuration** → **Static**:

| Device   | IP            | Mask          | Gateway      |
| -------- | ------------- | ------------- | ------------ |
| Server0  | 192.168.30.10 | 255.255.255.0 | 192.168.30.1 |
| Printer0 | 192.168.40.10 | 255.255.255.0 | 192.168.40.1 |

**Test from PC0:** `ping 192.168.30.10`

---

## Step 7 — DNS on Server (GUI)

1. **Server0** → **Services** → **DNS** → **On**
2. Add A record: `company.local` → `192.168.30.10`

**Test from PC0:** `ping company.local` (set DNS to `192.168.30.10` on PC if needed)

---

## Step 8 — HTTP on Server (GUI)

1. **Server0** → **Services** → **HTTP** → **On**
2. On PC0 → **Web Browser** → `http://company.local`

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

- From PC2 (Sales): `ping 192.168.10.10` — **fail**
- From PC0 (HR): `ping 192.168.20.10` — **work**

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

---

## Step 12 — Spanning Tree (optional read)

On SW1 or SW2:

```cisco
show spanning-tree
```

---

## Step 13 — Save Lab File

**Packet Tracer:** **File → Save As** your `.pkt` file.

(Optional final save on all three devices if you skipped `write memory` earlier: `enable` → `write memory`.)

---

## Done Checklist

| Test | Command | Should work? |
| ---- | ------- | ------------ |
| Same VLAN | PC0: `ping 192.168.10.11` | Yes |
| Cross VLAN | PC0: `ping 192.168.20.10` | Yes |
| Server | PC0: `ping 192.168.30.10` | Yes |
| DNS | PC0: `ping company.local` | Yes |
| ACL block | PC2: `ping 192.168.10.10` | No |
| Trunks | SW1: `show interfaces trunk` | Fa0/23, Fa0/24 |
| Router | R1: `show ip interface brief` | .10–.40 up/up |

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
