# CCNA Packet Tracer Lab – Layer 2 Security (DHCP Snooping, DAI & Port Security)

## Lab Overview

Your other labs build **connectivity** (VLANs, routing, OSPF, wireless). This lab builds **access-layer defenses** — the features that stop attacks *inside* the LAN before traffic ever hits a router ACL.

You will:

1. Build a small dual-VLAN LAN with router-on-a-stick + DHCP
2. Enable **DHCP Snooping** and prove a **rogue DHCP** Offer is dropped
3. Enable **Dynamic ARP Inspection (DAI)** and prove forged ARP is dropped
4. Add **Port Security** on a user port (MAC limit / violation)
5. (Stretch) Try **IP Source Guard** if your Packet Tracer image supports it

**Prerequisite:** VLANs, trunks, router-on-a-stick, and DHCP from the Small Office / ACL labs.

**Theory pair:** [Layer-2-Security-Study-Guide.md](../../security/Layer-2-Security-Study-Guide.md)

**Why this is new:** You already have notes on DHCP snooping / DAI / IPSG, but no dedicated hands-on lab. Port security appeared briefly in the small office lab — here it is part of a full L2 security story.

---

# Key Concepts (Quick Review)


| Feature            | Stops                                      | Depends on                         |
| ------------------ | ------------------------------------------ | ---------------------------------- |
| **DHCP Snooping**  | Rogue DHCP Offers / Acks on untrusted ports | Correct **trusted** ports          |
| **DAI**            | ARP spoofing (fake gateway MAC)            | **DHCP snooping binding table**    |
| **Port Security**  | Extra / unauthorized MACs on a port        | Sticky or static MAC limits        |
| **IP Source Guard**| Source IP spoof from an access port        | Bindings (DHCP or static)          |


**Mental model:** Access ports = **untrusted**. Path to the real DHCP server / gateway uplink = **trusted**.

**Safe enable order:** know the real DHCP path → enable snooping + trust → verify bindings → enable DAI → (optional) IPSG.

---

# Network Topology

```text
                         R1 (2911)
                    g0/0 — router-on-a-stick
                    .10 = 192.168.10.1  (VLAN 10)
                    .20 = 192.168.20.1  (VLAN 20)
                    DHCP pools for both VLANs
                              |
                         Trunk Fa0/24
                              |
                           SW1 (2960)
            +-----------------+------------------+
            |                 |                  |
         Fa0/1             Fa0/2              Fa0/3
         VLAN 10           VLAN 10            VLAN 20
            |                 |                  |
          PC0               ATTACKER           PC1
       (legit user)      (rogue DHCP /        (legit user)
                          ARP spoof later)
```

**Story:** PC0 and PC1 are normal clients. The **ATTACKER** PC will first run a fake DHCP server, then later send bad ARP. Your job is to configure SW1 so those attacks fail while legitimate DHCP still works.

Topology diagram: `CCNA_Layer2_Security_Topology.puml` (same folder).

---

# Devices Needed


| Device                         | Role                                      |
| ------------------------------ | ----------------------------------------- |
| 1× Router (2911) – `R1`        | Gateways + legitimate DHCP server         |
| 1× Switch (2960) – `SW1`       | Access switch; all L2 security features   |
| 2× PC – `PC0`, `PC1`           | Legitimate DHCP clients                   |
| 1× PC – `ATTACKER`             | Rogue DHCP / ARP spoof (Simulation tests) |


**Cabling (copper straight-through):**

| Link              | Ports              |
| ----------------- | ------------------ |
| R1 ↔ SW1          | G0/0 ↔ Fa0/24      |
| PC0 ↔ SW1         | Fa0 ↔ Fa0/1        |
| ATTACKER ↔ SW1    | Fa0 ↔ Fa0/2        |
| PC1 ↔ SW1         | Fa0 ↔ Fa0/3        |

---

# Address Plan


| VLAN | Name | Network           | Gateway (R1 subif) |
| ---- | ---- | ----------------- | ------------------ |
| 10   | USER | 192.168.10.0/24   | 192.168.10.1       |
| 20   | VOICE| 192.168.20.0/24   | 192.168.20.1       |
| 99   | NATIVE | (trunk native)  | —                  |


| Device   | VLAN | How it gets an IP                         |
| -------- | ---- | ----------------------------------------- |
| PC0      | 10   | DHCP from R1                              |
| ATTACKER | 10   | Static for attack tests (see steps below) |
| PC1      | 20   | DHCP from R1                              |
| R1       | —    | Subinterfaces `.10` / `.20` as above      |


**DHCP on R1:**

| Pool   | Network           | Default-router   | Exclude          |
| ------ | ----------------- | ---------------- | ---------------- |
| USER   | 192.168.10.0/24   | 192.168.10.1     | .1 – .10         |
| VOICE  | 192.168.20.0/24   | 192.168.20.1     | .1 – .10         |

---

# Step 1: Baseline — VLANs, Trunk, Access Ports

On **SW1**:

```cisco
enable
configure terminal
hostname SW1

vlan 10
 name USER
vlan 20
 name VOICE
vlan 99
 name NATIVE

interface fa0/1
 switchport mode access
 switchport access vlan 10
 no shutdown

interface fa0/2
 switchport mode access
 switchport access vlan 10
 no shutdown

interface fa0/3
 switchport mode access
 switchport access vlan 20
 no shutdown

interface fa0/24
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99
 no shutdown

end
write memory
```

Verify:

```cisco
show vlan brief
show interfaces trunk
```

**Expected — `show vlan brief`:**

```text
SW1# show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/4, Fa0/5, Fa0/6, Fa0/7
                                                Fa0/8, Fa0/9, Fa0/10, Fa0/11
                                                Fa0/12, Fa0/13, Fa0/14, Fa0/15
                                                Fa0/16, Fa0/17, Fa0/18, Fa0/19
                                                Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                Gig0/1, Gig0/2
10   USER                             active    Fa0/1, Fa0/2
20   VOICE                            active    Fa0/3
99   NATIVE                           active
1002 fddi-default                     active
1003 token-ring-default               active
1004 fddinet-default                  active
1005 trnet-default                    active
```

**What to notice:** Fa0/1 and Fa0/2 in VLAN **10**; Fa0/3 in VLAN **20**. Fa0/24 is a **trunk**, so it does **not** appear as an access port under a VLAN.

**Expected — `show interfaces trunk`:**

```text
SW1# show interfaces trunk

Port        Mode         Encapsulation  Status        Native vlan
Fa0/24      on           802.1q         trunking      99

Port        Vlans allowed on trunk
Fa0/24      10,20,99

Port        Vlans allowed and active in management domain
Fa0/24      10,20,99

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/24      10,20,99
```

**What to notice:** Status **trunking**, Native vlan **99**, allowed VLANs **10,20,99**.

---

# Step 2: Router-on-a-Stick + Legitimate DHCP

On **R1**:

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

ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.20.1 192.168.20.10

ip dhcp pool USER
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 192.168.10.1

ip dhcp pool VOICE
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 192.168.20.1

end
write memory
```

On **PC0** and **PC1**: Desktop → IP Configuration → **DHCP**.

Confirm:

- PC0 gets an address in `192.168.10.0/24`, gateway `192.168.10.1`
- PC1 gets an address in `192.168.20.0/24`, gateway `192.168.20.1`
- PC0 can ping `192.168.10.1` and `192.168.20.1` (cross-VLAN via R1)

**Expected — PC0 Command Prompt (`ipconfig`):**

```text
PC> ipconfig

IP Address......................: 192.168.10.11
Subnet Mask.....................: 255.255.255.0
Default Gateway.................: 192.168.10.1
```

**Expected — PC1 Command Prompt (`ipconfig`):**

```text
PC> ipconfig

IP Address......................: 192.168.20.11
Subnet Mask.....................: 255.255.255.0
Default Gateway.................: 192.168.20.1
```

(Exact host octets may be `.12`, `.13`, … if leases were renewed earlier — still must be outside the excluded `.1`–`.10` range.)

**Expected — PC0 ping gateway:**

```text
PC> ping 192.168.10.1

Reply from 192.168.10.1: bytes=32 time=1ms TTL=255
Reply from 192.168.10.1: bytes=32 time=1ms TTL=255
Reply from 192.168.10.1: bytes=32 time=1ms TTL=255
Reply from 192.168.10.1: bytes=32 time=1ms TTL=255
```

On R1:

```cisco
show ip dhcp binding
show ip interface brief
```

**Expected — `show ip interface brief` (R1):**

```text
R1# show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     unassigned      YES unset  up                    up
GigabitEthernet0/0.10  192.168.10.1    YES manual up                    up
GigabitEthernet0/0.20  192.168.20.1    YES manual up                    up
GigabitEthernet0/1     unassigned      YES unset  administratively down down
Vlan1                  unassigned      YES unset  administratively down down
```

**What to notice:** Parent `GigabitEthernet0/0` is **up/up** with no IP; both subinterfaces are **up/up** with the gateway addresses.

**Expected — `show ip dhcp binding` (R1):**

```text
R1# show ip dhcp binding
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.10.11    0060.XXXX.XXXX          --                      Automatic
192.168.20.11    0060.YYYY.YYYY          --                      Automatic
```

**What to notice:** One lease in `192.168.10.0/24` (PC0) and one in `192.168.20.0/24` (PC1). Client MAC/Client-ID values vary per PC NIC in Packet Tracer.

**Checkpoint:** Connectivity works *before* any security features. Do not enable snooping yet.

---

# Step 3: Demonstrate the Rogue DHCP Problem (Optional but Recommended)

Before defenses, show why snooping exists.

1. On **ATTACKER**, set a **static** IP in VLAN 10, e.g. `192.168.10.50/24`, gateway `192.168.10.1`.
2. In Packet Tracer, open ATTACKER → **Services** → **DHCP** → turn the service **On**.
   - Pool network: `192.168.10.0/24`
   - Default gateway: `192.168.10.50` (attacker as gateway — classic rogue)
   - Start IP: something like `192.168.10.100`
3. On **PC0**: release/renew DHCP (`ipconfig /release` then `/renew` in Command Prompt, or toggle DHCP off/on in the GUI).

**Observe:** PC0 may receive a lease from the **ATTACKER** (wrong gateway). That is the attack DHCP snooping is meant to stop.

**Expected — PC0 `ipconfig` if the rogue wins (before snooping):**

```text
PC> ipconfig

IP Address......................: 192.168.10.100
Subnet Mask.....................: 255.255.255.0
Default Gateway.................: 192.168.10.50
```

**What to notice:** Gateway is **`192.168.10.50`** (ATTACKER), not `192.168.10.1` (R1). Ping to other VLANs / “internet” style paths will break or get redirected.

4. Turn ATTACKER DHCP **Off** again and renew PC0 so it gets a clean lease from R1 before Step 4. Confirm gateway is back to `192.168.10.1`.

---

# Step 4: Enable DHCP Snooping

On **SW1**, enable snooping and **trust only the uplink toward R1** (the real DHCP server path). Leave Fa0/1–Fa0/3 **untrusted** (default).

```cisco
configure terminal

ip dhcp snooping
ip dhcp snooping vlan 10,20

! Packet Tracer / many IOS images insert Option 82; some DHCP servers
! drop those packets. If clients stop getting leases after snooping,
! try this next line (common lab fix):
no ip dhcp snooping information option

interface fa0/24
 ip dhcp snooping trust

end
```

**Do not** put `ip dhcp snooping trust` on Fa0/1, Fa0/2, or Fa0/3.

Verify:

```cisco
show ip dhcp snooping
show ip dhcp snooping binding
```

Renew DHCP on **PC0** and **PC1**. Bindings should appear for their ports.

**Expected — `show ip dhcp snooping`:**

```text
SW1# show ip dhcp snooping
Switch DHCP snooping is enabled
DHCP snooping is configured on following VLANs:
10,20
DHCP snooping is operational on following VLANs:
10,20
DHCP snooping is configured on the following L3 Interfaces:

Insertion of option 82 is disabled
Option 82 on untrusted port is not allowed
Verification of hwaddr field is enabled
Interface                    Trusted     Rate limit (pps)
------------------------     -------     ----------------
FastEthernet0/24             yes         unlimited
```

**What to notice:**

- Snooping **enabled** and operational on VLANs **10,20**
- Option 82 **disabled** (matches `no ip dhcp snooping information option`)
- Only **Fa0/24** is Trusted = **yes**
- Fa0/1–Fa0/3 do **not** appear as trusted (untrusted by default)

**Expected — `show ip dhcp snooping binding` (after PC0 and PC1 renew):**

```text
SW1# show ip dhcp snooping binding
MacAddress          IpAddress        Lease(sec)  Type           VLAN  Interface
------------------  ---------------  ----------  -------------  ----  --------------------
00:60:XX:XX:XX:AA   192.168.10.11    86390       dhcp-snooping  10    FastEthernet0/1
00:60:XX:XX:XX:BB   192.168.20.11    86395       dhcp-snooping  20    FastEthernet0/3
Total number of bindings: 2
```

**What to notice:**

| Client | VLAN | Interface | IP range |
| ------ | ---- | --------- | -------- |
| PC0    | 10   | **Fa0/1** | `192.168.10.11+` |
| PC1    | 20   | **Fa0/3** | `192.168.20.11+` |

ATTACKER on Fa0/2 should **not** have a `dhcp-snooping` binding unless it also got a DHCP lease from R1 (this lab uses static on ATTACKER).

*(Packet Tracer may show MAC format as `0060.XXXX.XXXX` instead of colon notation — same idea.)*

---

# Step 5: Prove Rogue DHCP Is Blocked

1. Turn **ATTACKER** DHCP service **On** again (same bad gateway as Step 3).
2. On **PC0**, release and renew DHCP.

**Expected:** PC0 still gets (or keeps) a lease from **R1**, not from ATTACKER. Offers from Fa0/2 are dropped because the port is **untrusted**.

**Expected — PC0 `ipconfig` after renew (snooping on, rogue DHCP on):**

```text
PC> ipconfig

IP Address......................: 192.168.10.11
Subnet Mask.....................: 255.255.255.0
Default Gateway.................: 192.168.10.1
```

**What to notice:** Gateway stays **`192.168.10.1`** (R1). Compare to Step 3 where it became `.50`.

3. On SW1, re-check:

```cisco
show ip dhcp snooping binding
```

**Expected — binding still correct (rogue did not win):**

```text
SW1# show ip dhcp snooping binding
MacAddress          IpAddress        Lease(sec)  Type           VLAN  Interface
------------------  ---------------  ----------  -------------  ----  --------------------
00:60:XX:XX:XX:AA   192.168.10.11    86400       dhcp-snooping  10    FastEthernet0/1
00:60:XX:XX:XX:BB   192.168.20.11    86390       dhcp-snooping  20    FastEthernet0/3
Total number of bindings: 2
```

Binding for PC0 should still map to **Fa0/1**, with an IP from R1’s pool (typically `.11`+ given your exclusions). There should be **no** binding that points PC0 at gateway `.50` or at Fa0/2 as the client port for a “good” lease from the attacker.

4. Turn ATTACKER DHCP **Off** when done.

**What you learned:** Untrusted ports may send Discover/Request; server messages (Offer/Ack) from them are dropped.

---

# Step 6: Enable Dynamic ARP Inspection (DAI)

DAI validates ARP on untrusted ports against the **snooping binding table**. Trust the same uplink you trusted for DHCP.

```cisco
configure terminal

ip arp inspection vlan 10,20

interface fa0/24
 ip arp inspection trust

end
```

Verify:

```cisco
show ip arp inspection
show ip arp inspection statistics
show ip dhcp snooping binding
```

**Expected — `show ip arp inspection`:**

```text
SW1# show ip arp inspection

Source Mac Validation      : Disabled
Destination Mac Validation : Disabled
IP Address Validation      : Disabled

Vlan     Configuration    Operation   ACL Match          Static ACL
----     -------------    ---------   ---------          ----------
10       Enabled          Active
20       Enabled          Active

Vlan     ACL Logging      DHCP Logging      Probe Logging
----     -----------      ------------      -------------
10       Deny             Deny              Off
20       Deny             Deny              Off

Vlan      Forwarded        Dropped     DHCP Drops      ACL Drops
----      ---------        -------     ----------      ---------
10              0              0              0              0
20              0              0              0              0

Vlan   DHCP Permits    ACL Permits   Probe Permits   Source MAC Failures
----   ------------    -----------   -------------   -------------------
10              0              0               0                     0
20              0              0               0                     0

Vlan   Dest MAC Failures      IP Validation Failures   Invalid Protocol Data
----   -----------------      ----------------------   ---------------------
10                   0                        0                        0
20                   0                        0                        0
```

*(Packet Tracer may show a shorter form. Minimum you need: VLANs **10** and **20** = Enabled / Active.)*

Also confirm the uplink is trusted for DAI:

```cisco
show running-config interface fa0/24
```

**Expected snippet:**

```text
interface FastEthernet0/24
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99
 switchport mode trunk
 ip dhcp snooping trust
 ip arp inspection trust
```

**Expected — `show ip arp inspection statistics` (healthy baseline, after a few legitimate pings):**

```text
SW1# show ip arp inspection statistics

Vlan      Forwarded        Dropped     DHCP Drops      ACL Drops
----      ---------        -------     ----------      ---------
10              4              0              0              0
20              2              0              0              0
```

**What to notice:** **Forwarded** increases when PC0/PC1 ARP normally. **Dropped** stays **0** until a spoof attempt.

Generate a little ARP with:

```text
PC> ping 192.168.10.1
```

Then re-check statistics — Forwarded on VLAN 10 should rise; Dropped should still be 0.

**Dependency check:** If bindings are empty (clients on static IPs with no static ARP ACL), DAI will drop their ARP and break connectivity. This lab keeps PC0/PC1 on DHCP on purpose. Confirm bindings still look like Step 4 before trusting DAI.

---

# Step 7: Prove ARP Spoofing Is Dropped (Simulation / Manual Check)

Packet Tracer support for live ARP spoof tools varies by version. Use whichever works on your build:

### Option A — Simulation mode

1. Open **Simulation** mode.
2. Filter for **ARP** (and ICMP if useful).
3. From PC0, `ping 192.168.10.1` and confirm ARP request/reply succeeds (legitimate).
4. Mentally map: an ARP reply from ATTACKER claiming `192.168.10.1` with ATTACKER’s MAC would **not** match the binding for Fa0/2 → DAI drops it on an untrusted port.

### Option B — Force a mismatch (if your PT build allows)

Some builds let you send gratuitous ARP from ATTACKER claiming the gateway IP. After attempting:

```cisco
show ip arp inspection statistics
```

**Expected after a spoof attempt (VLAN 10 Dropped increases):**

```text
SW1# show ip arp inspection statistics

Vlan      Forwarded        Dropped     DHCP Drops      ACL Drops
----      ---------        -------     ----------      ---------
10              6              3              3              0
20              2              0              0              0
```

**What to notice:** **Dropped** / **DHCP Drops** on VLAN **10** go up. Exact counts vary; the key is Dropped is **no longer 0** while legitimate Forwarded still works for PC0.

### Option C — Break it on purpose (learn the footgun)

Temporarily remove the binding path:

```cisco
configure terminal
no ip dhcp snooping
! or shut Fa0/24 trust wrongly — then restore
```

Renew clients, re-enable snooping + trust, rebuild bindings, then re-enable DAI. Document what broke and why.

**CCNA takeaway:** DAI without a trustworthy binding source is a LAN outage waiting to happen.

---

# Step 8: Port Security on a User Port

Complementary defense: limit how many MACs Fa0/1 will learn.

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

Have PC0 generate traffic (`ping 192.168.10.1`) so the sticky MAC is learned.

Verify:

```cisco
show port-security interface fa0/1
show port-security address
```

**Expected — `show port-security interface fa0/1` (after PC0 traffic):**

```text
SW1# show port-security interface fa0/1
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Shutdown
Maximum MAC Addresses      : 1
Total MAC Addresses        : 1
Configured MAC Addresses   : 0
Sticky MAC Addresses       : 1
Last Source Address:Vlan   : 0060.XXXX.XXXX:10
Security Violation Count   : 0
```

**What to notice:** **Enabled**, **Secure-up**, Maximum **1**, Sticky **1**, Violation Count **0**. Last Source Address matches PC0’s NIC MAC.

**Expected — `show port-security address`:**

```text
SW1# show port-security address
          Secure Mac Address Table
-----------------------------------------------------------------------------
Vlan    Mac Address       Type                Ports   Remaining Age
                                                         (mins)
----    -----------       ----                -----   -------------
  10    0060.XXXX.XXXX    SecureSticky        Fa0/1        -
-----------------------------------------------------------------------------
Total Addresses: 1
```

**Violation test:**

1. Disconnect PC0 from Fa0/1.
2. Connect **ATTACKER** (or another PC) to Fa0/1 instead.
3. Generate traffic from the new device.

**Expected:** Port goes **err-disabled** (violation shutdown).

**Expected — `show port-security interface fa0/1` after violation:**

```text
SW1# show port-security interface fa0/1
Port Security              : Enabled
Port Status                : Secure-shutdown
Violation Mode             : Shutdown
Maximum MAC Addresses      : 1
Total MAC Addresses        : 1
Configured MAC Addresses   : 0
Sticky MAC Addresses       : 1
Last Source Address:Vlan   : 0060.YYYY.YYYY:10
Security Violation Count   : 1
```

**What to notice:** Port Status = **Secure-shutdown** (not Secure-up). Violation Count ≥ **1**. Last Source Address is the **unauthorized** MAC.

**Expected — `show interfaces status` (or `show interfaces fa0/1 status`):**

```text
SW1# show interfaces status

Port      Name               Status       Vlan       Duplex  Speed Type
Fa0/1                        err-disabled 10         auto    auto  10/100BaseTX
Fa0/2                        connected    10         a-full  a-100 10/100BaseTX
Fa0/3                        connected    20         a-full  a-100 10/100BaseTX
Fa0/24                       connected    trunk      a-full  a-100 10/100BaseTX
```

**What to notice:** Fa0/1 = **err-disabled**.

Recover:

```cisco
configure terminal
interface fa0/1
 shutdown
 no shutdown
end
```

Reconnect the legitimate PC0. Confirm sticky MAC and DHCP still work.

**Expected after recovery (PC0 back on Fa0/1):**

```text
SW1# show port-security interface fa0/1
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Shutdown
Maximum MAC Addresses      : 1
Total MAC Addresses        : 1
Sticky MAC Addresses       : 1
Security Violation Count   : 1
```

Port Status returns to **Secure-up**. Violation Count may still show the earlier hit until you clear counters / reload depending on platform — the important part is the port forwards again for the sticky MAC.

Optional softer mode for labs (less dramatic):

```cisco
switchport port-security violation restrict
```

Then watch `show port-security interface fa0/1` for **Security Violation Count** instead of err-disable — Port Status stays **Secure-up**, but the count increases and unauthorized frames are dropped.

---

# Step 9: End-to-End Verification Checklist


| Check | Command / action | Pass if (see expected outputs above) |
| ----- | ---------------- | ------------------------------------ |
| VLANs | `show vlan brief` | Fa0/1–2 = VLAN 10; Fa0/3 = VLAN 20; VLAN 99 exists |
| Trunk | `show interfaces trunk` | Fa0/24 **trunking**, native **99**, allowed **10,20,99** |
| DHCP (R1) | `show ip dhcp binding` | Leases for `.10.x` and `.20.x` from R1 |
| DHCP (PC) | `ipconfig` on PC0/PC1 | Gateways `.10.1` / `.20.1` |
| Snooping | `show ip dhcp snooping` | Enabled on 10,20; **only Fa0/24** trusted |
| Bindings | `show ip dhcp snooping binding` | PC0→**Fa0/1**, PC1→**Fa0/3** |
| Rogue DHCP | ATTACKER DHCP on; PC0 renew | Gateway still **192.168.10.1** (not `.50`) |
| DAI | `show ip arp inspection` | 10,20 Enabled/Active; Fa0/24 has `ip arp inspection trust` |
| DAI stats | `show ip arp inspection statistics` | Forwarded > 0 after pings; Dropped rises only on spoof |
| Reachability | PC0 `ping 192.168.10.1` / `ping 192.168.20.11` | Replies |
| Port security | `show port-security interface fa0/1` | Enabled, **Secure-up**, max 1, sticky 1 |
| Violation | Swap host on Fa0/1 | **Secure-shutdown** / **err-disabled** |

---

# Troubleshooting Challenges

Practice one break at a time.

## Scenario 1 – Clients Get No DHCP After Enabling Snooping

Usually: forgot `ip dhcp snooping trust` on Fa0/24, or Option 82 issue.

```cisco
show ip dhcp snooping
interface fa0/24
 ip dhcp snooping trust
!
no ip dhcp snooping information option
```

## Scenario 2 – Trusted the Attacker Port by Mistake

```cisco
interface fa0/2
 no ip dhcp snooping trust
```

Rogue Offers start working again until you remove trust.

## Scenario 3 – DAI Drops Everything

Bindings empty (static clients, or snooping never built table). Fix DHCP/snooping first; only then enable DAI.

```cisco
show ip dhcp snooping binding
show ip arp inspection statistics
```

## Scenario 4 – Port Err-Disabled After Port Security

```cisco
show interfaces status
show port-security interface fa0/1
interface fa0/1
 shutdown
 no shutdown
```

## Scenario 5 – Cross-VLAN Ping Fails but Same-VLAN Works

Router subinterface or trunk problem — not an L2 security feature. Check R1 `show ip interface brief` and SW1 trunk.

---

# Stretch Goals

1. **IP Source Guard** on Fa0/1 (if supported in your PT version):

```cisco
interface fa0/1
 ip verify source
! or, on some platforms:
! ip verify source port-security
```

Then try setting PC0 to a **stolen** static IP that belongs to another binding and see traffic fail. Verify with `show ip verify source`.

2. Add a **static printer** on VLAN 10 that does not use DHCP — create a **static DHCP snooping binding** (syntax varies; document what your image allows) before enabling IPSG/DAI for that host.

3. Move the DHCP server off R1 onto a **Server** in VLAN 10, and figure out which ports must be **trusted** (path from client → switch → server). Wrong trust = broken DHCP or open rogue path.

4. Draw the enable-order flowchart from the study guide from memory, then compare.

---

# Skills Learned

- Trusted vs untrusted ports on the access layer
- DHCP snooping to block rogue DHCP and build bindings
- Dynamic ARP Inspection using those bindings
- Port security (sticky MAC, violation shutdown)
- Safe order of enabling L2 security features
- Distinguishing L2 attacks from routing/ACL problems

**Maps to CCNA domains:** Network Access (switch security), Security Fundamentals, IP Services (DHCP).

---

# Common Mistakes


| Mistake | Fix |
| ------- | --- |
| Trusting all access ports “so users work” | Only trust infrastructure / real DHCP path |
| Enabling DAI before bindings exist | Snooping + successful leases first |
| Forgetting Option 82 quirk after snooping | `no ip dhcp snooping information option` |
| Expecting IPSG to allow static hosts automatically | Add static bindings or use DHCP |
| Confusing port security with DAI | Port security = MAC count; DAI = ARP vs binding |

---

# What to Save

- Packet Tracer file: `Layer2_Security_DHCP_Snooping_DAI.pkt` (optional, in this folder)
- Screenshot of topology with trusted uplink labeled
- Output: `show ip dhcp snooping binding`
- Output: `show ip arp inspection` / statistics after a test
- Output: `show port-security interface fa0/1`
- Short note: what happened when rogue DHCP was on **before** vs **after** snooping

---

# Reflection Prompts

After you finish, jot answers in your [README](../../README.md) journal:

1. Why must the port toward R1 be trusted for both snooping and DAI?
2. What table does DAI consult, and who builds it?
3. If ATTACKER and PC0 swap cables, which feature(s) catch that — snooping, DAI, port security, or more than one?
