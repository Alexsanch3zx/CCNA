# CCNA Packet Tracer Lab – Small Office Network (Complete Step-by-Step)

Use this document **in order**. Do not skip wiring or trunk steps—inter-VLAN routing will fail without them.

**Tool:** Cisco Packet Tracer  
**Time:** ~2–4 hours first time

---

## What You Will Build

A small office with **4 VLANs**, **two switches**, **one router** (router-on-a-stick), **DHCP**, **DNS**, **HTTP** on a server, an **ACL** blocking Sales → HR, and **port security** on one port.

---

## Part A — Reference Tables (Keep Open While You Work)

### VLANs and networks


| VLAN | Name     | Network         | Gateway (router) |
| ---- | -------- | --------------- | ---------------- |
| 10   | HR       | 192.168.10.0/24 | 192.168.10.1     |
| 20   | Sales    | 192.168.20.0/24 | 192.168.20.1     |
| 30   | Servers  | 192.168.30.0/24 | 192.168.30.1     |
| 40   | Printers | 192.168.40.0/24 | 192.168.40.1     |


### Device IPs (after DHCP or static)


| Device   | VLAN | IP (static option) | Gateway      |
| -------- | ---- | ------------------ | ------------ |
| PC0      | 10   | 192.168.10.10      | 192.168.10.1 |
| PC1      | 10   | 192.168.10.11      | 192.168.10.1 |
| PC2      | 20   | 192.168.20.10      | 192.168.20.1 |
| PC3      | 20   | 192.168.20.11      | 192.168.20.1 |
| Server0  | 30   | 192.168.30.10      | 192.168.30.1 |
| Printer0 | 40   | 192.168.40.10      | 192.168.40.1 |


**Plan:** PCs use **DHCP** after Step 8. Server and printer use **static** IPs (Step 7).

### DHCP pools (router) — exclude static hosts


| Pool name | Network      | Default router | Excluded (do not assign)   |
| --------- | ------------ | -------------- | -------------------------- |
| HR        | 192.168.10.0 | 192.168.10.1   | .10, .11 (optional static) |
| SALES     | 192.168.20.0 | 192.168.20.1   | .10, .11                   |
| SERVERS   | 192.168.30.0 | 192.168.30.1   | .10 (server)               |
| PRINTERS  | 192.168.40.0 | 192.168.40.1   | .10 (printer)              |


---

## Part B — Topology and Cabling (Packet Tracer GUI)

### Devices to add


| Qty | Device type in PT        | Rename (optional) |
| --- | ------------------------ | ----------------- |
| 1   | Router → **2911**        | R1                |
| 2   | Switch → **2960**        | SW1, SW2          |
| 4   | PC                       | PC0–PC3           |
| 1   | Server                   | Server0           |
| 1   | Printer (or generic end) | Printer0          |


### Cable type

Use **Copper Straight-Through** (green solid line) for all Ethernet links in this lab.

### Exact connections


| From device | From port | To device | To port | Notes                    |
| ----------- | --------- | --------- | ------- | ------------------------ |
| R1          | Gig0/0    | SW1       | Fa0/24  | Router-on-a-stick uplink |
| SW1         | Fa0/1     | PC0       | Fa0     | HR                       |
| SW1         | Fa0/2     | PC1       | Fa0     | HR                       |
| SW1         | Fa0/23    | SW2       | Fa0/23  | Inter-switch trunk       |
| SW2         | Fa0/1     | PC2       | Fa0     | Sales                    |
| SW2         | Fa0/2     | PC3       | Fa0     | Sales                    |
| SW2         | Fa0/3     | Server0   | Fa0     | Servers VLAN             |
| SW2         | Fa0/4     | Printer0  | Fa0     | Printers VLAN            |


### Topology diagram

```text
                    [ R1 Gi0/0 ]
                          |
                    [ SW1 Fa0/24 ]
                    Fa0/1   Fa0/2
                     |       |
                   PC0     PC1
                    |
              [ SW1 Fa0/23 ]----[ SW2 Fa0/23 ]
                                    |
                    Fa0/1  Fa0/2  Fa0/3  Fa0/4
                     |      |      |       |
                    PC2    PC3  Server0  Printer0
```

### After cabling — wait for link lights

Click **Fast Forward Time** until router/switch interfaces show **up/up** (or configure `no shutdown` in CLI).

---

## Part C — Access the CLI in Packet Tracer

1. Click **SW1** (or R1).
2. Open the **CLI** tab.
3. Press **Enter** if asked to terminate autoinstall — choose **no** if prompted.
4. You should see `Switch>` or `Router>`.

**Tip:** Type `enable` then `configure terminal` (abbrev: `conf t`) before pasting configs below.

---

## Step 1 — Create VLANs (Both Switches)

**Where:** SW1 **and** SW2 (run the **same** commands on each).

**Why:** VLANs must exist on every switch that carries those VLANs over a trunk.

### SW1 and SW2 — identical VLAN block

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
```

### Verify (both switches)

```cisco
show vlan brief
```

You should see VLANs **10, 20, 30, 40** in the list (plus default VLAN 1).

---

## Step 2 — Assign Access Ports (Each Switch Separately)

**Where:** Ports differ per switch — do **not** copy SW1 port numbers onto SW2.

### SW1 only

```cisco
enable
configure terminal

# Selects multiple interfaces at once:Fa0/1 and Fa0/2
interface range fa0/1-2

 # Sets the selected ports as access ports.
 switchport mode access

 # Assigns the access ports to VLAN 10.
 # After this command: Fa0/1 → VLAN 10 and Fa0/2 → VLAN 10
 switchport access vlan 10

# Leaves Interface Configuration mode and returns to Global Configuration mode.
exit

# Returns directly to Privileged EXEC mode.
end
```


| Port  | Device | VLAN |
| ----- | ------ | ---- |
| Fa0/1 | PC0    | 10   |
| Fa0/2 | PC1    | 10   |


### SW2 only

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
```


| Port  | Device   | VLAN |
| ----- | -------- | ---- |
| Fa0/1 | PC2      | 20   |
| Fa0/2 | PC3      | 20   |
| Fa0/3 | Server0  | 30   |
| Fa0/4 | Printer0 | 40   |


### Verify

On each switch:

```cisco
show vlan brief
```

Confirm access ports appear under the correct VLAN (not VLAN 1).

---

## Step 3 — Trunk: SW1 ↔ SW2

**Where:** SW1 **and** SW2 on **Fa0/23**.

### SW1

```cisco
enable
configure terminal

interface fa0/23
 switchport mode trunk
exit

end
```

### SW2

```cisco
enable
configure terminal

interface fa0/23
 switchport mode trunk
exit

end
```

### Verify (either switch)

```cisco
show interfaces trunk
```

Expect **Fa0/23** in **trunking** state with VLANs **10, 20, 30, 40** allowed (default allows all on 2960 in PT).

---

## Step 4 — Trunk: SW1 ↔ Router (Critical)

**Where:** SW1 **Fa0/24** and R1 **Gig0/0**.

Without this trunk, router subinterfaces never receive tagged VLAN traffic.

### SW1

```cisco
enable
configure terminal

interface fa0/24
 switchport mode trunk
exit

end
```

### Verify SW1

```cisco
show interfaces trunk
```

You should see **Fa0/23** and **Fa0/24** as trunks.

---

## Step 5 — Router-on-a-Stick (R1 Only)

**Where:** Router **R1** only.

### 5.1 Enable physical interface

```cisco
enable
configure terminal

hostname R1

interface g0/0
 no shutdown
exit
```

### 5.2 Subinterfaces (one per VLAN)

```cisco
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
```

### 5.3 Verify router

```cisco
show ip interface brief
```

Expect **Gig0/0** up and **Gig0/0.10–.40** with correct IPs, status **up/up**.

### 5.4 First connectivity test (static IPs temporarily)

Before DHCP, set **one PC** manually to test routing:

**PC0 (Desktop → IP Configuration):**


| Field           | Value         |
| --------------- | ------------- |
| IP Address      | 192.168.10.10 |
| Subnet Mask     | 255.255.255.0 |
| Default Gateway | 192.168.10.1  |


**PC2:**


| Field           | Value         |
| --------------- | ------------- |
| IP Address      | 192.168.20.10 |
| Subnet Mask     | 255.255.255.0 |
| Default Gateway | 192.168.20.1  |


From **PC0** → **Command Prompt**:

```text
ping 192.168.20.10
```

**Expected:** replies (inter-VLAN via router).  
**If fails:** check trunks (Steps 3–4), subinterfaces (Step 5), gateways on PCs.

---

## Step 6 — Static IP: Server and Printer (Packet Tracer GUI)

**Where:** Server0 and Printer0 only (not PCs yet if you will use DHCP next).

### Server0

1. Click **Server0** → **Desktop** tab → **IP Configuration**.
2. Select **Static**.


| Field           | Value         |
| --------------- | ------------- |
| IP Address      | 192.168.30.10 |
| Subnet Mask     | 255.255.255.0 |
| Default Gateway | 192.168.30.1  |


### Printer0


| Field           | Value         |
| --------------- | ------------- |
| IP Address      | 192.168.40.10 |
| Subnet Mask     | 255.255.255.0 |
| Default Gateway | 192.168.40.1  |


### Test from PC0

```text
ping 192.168.30.10
ping 192.168.40.10
```

---

## Step 7 — DNS Service on Server (Packet Tracer GUI)

**Where:** Server0 — no CLI required.

1. Click **Server0** → **Services** tab → **DNS**.
2. Set **DNS Service** to **On**.
3. Under **DNS Records**, add:


| Name            | Type     | Address       |
| --------------- | -------- | ------------- |
| `company.local` | A Record | 192.168.30.10 |


(If PT asks for only hostname, use `company.local` pointing to **192.168.30.10**.)

### Test from PC0 (must have IP + gateway set)

**Desktop → Command Prompt:**

```text
ping company.local
```

**Expected:** resolves to 192.168.30.10 and pings succeed.

**If name fails:** On PC0, set **DNS Server** to `192.168.30.10` in IP Configuration, or use router as DNS forwarder in advanced labs—for this lab, set PC DNS to **192.168.30.10** after DHCP (Step 8).

---

## Step 8 — HTTP Service on Server (Packet Tracer GUI)

1. **Server0** → **Services** → **HTTP** → **On**.
2. Optional: edit index page text to `Welcome to Company Network`.
3. From **PC0** → **Desktop** → **Web Browser** → URL:

```text
http://192.168.30.10
```

or

```text
http://company.local
```

**Expected:** page loads.

---

## Step 9 — DHCP on Router (R1)

**Where:** R1 only.

### 9.1 Exclude static addresses

```cisco
enable
configure terminal

ip dhcp excluded-address 192.168.10.10 192.168.10.11
ip dhcp excluded-address 192.168.20.10 192.168.20.11
ip dhcp excluded-address 192.168.30.10
ip dhcp excluded-address 192.168.40.10
```

### 9.2 Create pools

```cisco
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
```

### 9.3 Set all four PCs to DHCP

For **PC0, PC1, PC2, PC3**:

1. **Desktop** → **IP Configuration**.
2. Select **DHCP**.
3. Wait a few seconds (or fast-forward time).

### 9.4 Verify DHCP

On each PC → **Desktop** → **Command Prompt**:

```text
ipconfig
```

Check:

- IP in correct subnet (10.x, 20.x, etc.).  
- Default gateway = `.1` for that VLAN.  
- DNS = `192.168.30.10` (if shown).

On **R1**:

```cisco
show ip dhcp binding
```

---

## Step 10 — ACL: Block Sales from HR

**Where:** R1 — applied **inbound** on subinterface **g0/0.20** (traffic **from** Sales VLAN entering the router).

### 10.1 Create ACL

```cisco
enable
configure terminal

access-list 100 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
access-list 100 permit ip any any

interface g0/0.20
 ip access-group 100 in

end
```

### 10.2 Test

From **PC2** or **PC3** (Sales):

```text
ping 192.168.10.10
```

**Expected:** **fail** (or timeout).

From **PC0** (HR):

```text
ping 192.168.20.10
```

**Expected:** **success** (ACL is one-directional for this rule).

From **Sales**:

```text
ping 192.168.30.10
```

**Expected:** **success** (HR block only).

---

## Step 11 — Port Security (SW1 Fa0/1)

**Where:** SW1 — port connected to **PC0**.

```cisco
enable
configure terminal

interface fa0/1
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
exit

end
```

### Verify

```cisco
show port-security interface fa0/1
```

### Optional test

Disconnect PC0, connect a **different** PC to Fa0/1 — port may go **err-disabled**. Recover:

```cisco
interface fa0/1
 shutdown
 no shutdown
```

---

## Step 12 — Spanning Tree (Investigation)

**Where:** SW1 or SW2 — read-only.

```cisco
show spanning-tree
```

Record on paper:


| Question              | Your answer (from output)     |
| --------------------- | ----------------------------- |
| Which switch is root? | Lowest priority / MAC wins    |
| Root port on SW2?     | Port toward root bridge       |
| Any blocking port?    | On redundant paths if present |


This lab has **no loop** (tree topology)—most ports will be **forwarding**. The exercise is learning to **read** the output.

---

## Step 13 — Save Configuration

**On R1, SW1, and SW2:**

```cisco
enable
write memory
```

Or:

```cisco
copy running-config startup-config
```

Press **Enter** when prompted for destination filename.

**In Packet Tracer:** **File → Save As** your `.pkt` file.

---

## Part D — Full Verification Checklist

Run these before you consider the lab done.


| #   | Test             | From | Command / action               | Expected             |
| --- | ---------------- | ---- | ------------------------------ | -------------------- |
| 1   | Same VLAN        | PC0  | `ping 192.168.10.11`           | Success              |
| 2   | Cross VLAN       | PC0  | `ping 192.168.20.10`           | Success              |
| 3   | To server        | PC0  | `ping 192.168.30.10`           | Success              |
| 4   | DNS              | PC0  | `ping company.local`           | Success              |
| 5   | HTTP             | PC0  | Browser `http://company.local` | Page loads           |
| 6   | ACL block        | PC2  | `ping 192.168.10.10`           | Fail                 |
| 7   | ACL allow server | PC2  | `ping 192.168.30.10`           | Success              |
| 8   | Trunk            | SW1  | `show interfaces trunk`        | Fa0/23, Fa0/24       |
| 9   | VLANs            | SW1  | `show vlan brief`              | Ports in 10,20,30,40 |
| 10  | Router IFs       | R1   | `show ip interface brief`      | .10–.40 up           |


---

## Part E — Troubleshooting Guide


| Symptom                       | Likely cause                       | Fix                                  |
| ----------------------------- | ---------------------------------- | ------------------------------------ |
| No link on router             | `g0/0` shutdown                    | `no shutdown`                        |
| PCs same VLAN can’t ping      | Wrong VLAN on port                 | `show vlan brief`, fix access VLAN   |
| PCs different VLAN can’t ping | Missing router subif or trunk      | Steps 4–5, `show ip int brief`       |
| Only SW2 VLANs fail           | Trunk Fa0/23 down / not trunk      | `show interfaces trunk`              |
| DHCP fails                    | Pools missing or wrong GW          | Step 9, `show ip dhcp pool`          |
| DNS name fails                | DNS off or PC DNS blank            | Server DNS on; set DNS 192.168.30.10 |
| Sales can ping HR             | ACL not applied or wrong direction | `show access-lists`, `g0/0.20 in`    |
| Port security err-disabled    | Wrong MAC on Fa0/1                 | `shutdown` / `no shutdown` on port   |


### Instructor-style break scenarios (practice)

1. **Wrong gateway on PC0** → set gateway to `192.168.10.2` and fix.
2. **PC2 on VLAN 10 instead of 20** → fix access port on SW2 Fa0/1.
3. **Trunk shut down** → `interface fa0/23` / `no shutdown`.
4. **ACL too broad** → remove `deny` or add `permit` as needed.

---

## Part F — Quick Command Reference

### Switch

```cisco
show vlan brief
show interfaces trunk
show port-security
show spanning-tree
show running-config
```

### Router

```cisco
show ip interface brief
show ip route
show ip dhcp binding
show access-lists
show running-config
```

---

## Skills Covered

- VLAN creation and access ports  
- Trunking (switch–switch and switch–router)  
- Router-on-a-stick inter-VLAN routing  
- DHCP and static addressing  
- DNS and HTTP services in Packet Tracer  
- Extended ACLs and direction (`in` on subinterface)  
- Port security  
- Spanning tree output interpretation  
- Structured verification and troubleshooting

---

**You are done when** every row in **Part D — Full Verification Checklist** passes.