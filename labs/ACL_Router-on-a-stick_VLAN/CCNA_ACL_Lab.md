# CCNA Lab: VLANs, Router-on-a-Stick, and ACL Security

## Topology

### Devices


| Device | Model (example) | Role |
| ------ | --------------- | ---- |
| R1 | Router 2911 | Inter-VLAN routing (router-on-a-stick) + ACL |
| SW1 | Switch 2960 | VLANs, access ports, trunk to R1 |
| PC0 | PC | HR host (VLAN 10) |
| PC1 | PC | SALES host (VLAN 20) |
| Server0 | Server | SERVERS host (VLAN 30) |


### Cabling (build this first)

Use **Copper Straight-Through** for every link (PC/server ↔ switch, and switch ↔ router in Packet Tracer).

| From | Port | To | Port | Cable | Purpose |
| ---- | ---- | -- | ---- | ----- | ------- |
| PC0 | FastEthernet0 | SW1 | FastEthernet0/1 | Straight-Through | HR access port → VLAN 10 |
| PC1 | FastEthernet0 | SW1 | FastEthernet0/2 | Straight-Through | SALES access port → VLAN 20 |
| Server0 | FastEthernet0 | SW1 | FastEthernet0/3 | Straight-Through | SERVERS access port → VLAN 30 |
| SW1 | FastEthernet0/24 | R1 | GigabitEthernet0/0 | Straight-Through | Trunk carrying VLANs 10, 20, 30 |


```text
                    R1 (2911)
                 GigabitEthernet0/0
                         |
                    (trunk link)
                         |
                 SW1 FastEthernet0/24
                         |
        +----------------+----------------+
        |                |                |
     Fa0/1            Fa0/2            Fa0/3
   (VLAN 10)        (VLAN 20)        (VLAN 30)
        |                |                |
       PC0              PC1            Server0
       (HR)           (SALES)         (SERVERS)
```

**How to read the diagram**

- Each end device plugs into **one access port** on SW1. That port belongs to only one VLAN.
- SW1 ↔ R1 is a **single physical cable**, but it is configured as a **trunk** so tagged frames for VLANs 10, 20, and 30 share that link.
- On R1, one physical interface (`G0/0`) is split into **subinterfaces** (`G0/0.10`, `G0/0.20`, `G0/0.30`) — that is router-on-a-stick. There is **no** separate cable per VLAN to the router.

**Packet Tracer tip:** After cabling, link lights should turn green once both ends are up (`no shutdown` on R1 `G0/0` and the switch ports). Amber/black usually means wrong ports, wrong cable type, or an interface still shut.

### Address plan (for reference while cabling)


| Device | VLAN | Switch port | IP | Mask | Gateway |
| ------ | ---- | ----------- | -- | ---- | ------- |
| PC0 | 10 HR | Fa0/1 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| PC1 | 20 SALES | Fa0/2 | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 |
| Server0 | 30 SERVERS | Fa0/3 | 192.168.30.10 | 255.255.255.0 | 192.168.30.1 |
| R1 | — | via Fa0/24 trunk | .1 on each subnet | — | — |


## Step 0: Cable the Lab

1. Place R1, SW1, PC0, PC1, and Server0 on the workspace.
2. Connect the four links using the cabling table above (click **Connections** → **Copper Straight-Through**).
3. Double-check port labels: hosts on **Fa0/1–Fa0/3**, router on **Fa0/24 ↔ G0/0** — do not put the router on an access port by mistake.
4. Leave interfaces default for now; you will configure VLANs and the trunk in the next steps.

## Step 1: Create VLANs

``` bash
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
```

## Step 2: Assign Ports to VLANs

``` bash
interface fa0/1
switchport mode access
switchport access vlan 10
exit

interface fa0/2
switchport mode access
switchport access vlan 20
exit

interface fa0/3
switchport mode access
switchport access vlan 30
exit
```

## Step 3: Configure Trunk Port

``` bash
interface fa0/24
switchport mode trunk
exit
```

## Step 4: Save Configuration

``` bash
end
write memory
```

## Step 5: Configure Router-on-a-Stick

``` bash
enable
configure terminal

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
```

## Step 6: Save Router

``` bash
end
write memory
```

## Step 7: Configure End Devices

### PC0

-   IP: 192.168.10.10
-   Mask: 255.255.255.0
-   Gateway: 192.168.10.1

### PC1

-   IP: 192.168.20.10
-   Mask: 255.255.255.0
-   Gateway: 192.168.20.1

### Server0

-   IP: 192.168.30.10
-   Mask: 255.255.255.0
-   Gateway: 192.168.30.1

## Step 8: Verification

``` bash
show vlan brief
show interfaces trunk
show ip interface brief
show ip route
```

## Step 9: Connectivity Tests

``` bash
ping 192.168.20.10
ping 192.168.30.10
```

``` bash
ping 192.168.10.10
ping 192.168.30.10
```

## Step 10: Create ACL

Goal: - HR → Server = Allowed - SALES → Server = Blocked

``` bash
configure terminal

access-list 101 deny ip 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255
access-list 101 permit ip any any

interface g0/0.20
ip access-group 101 in
exit
```

## Step 11: Save Configuration

``` bash
end
write memory
```

## Step 12: Verify ACL

``` bash
show access-lists
show ip interface g0/0.20
```

## Final Tests

From HR:

``` bash
ping 192.168.30.10
```

Expected: Success

From SALES:

``` bash
ping 192.168.30.10
```

Expected: Fail

From SALES:

``` bash
ping 192.168.10.10
```

Expected: Success

## Learning Objectives

-   Configure VLANs
-   Configure trunk links
-   Configure Router-on-a-Stick
-   Apply standard ACL security policies
-   Verify and troubleshoot inter-VLAN routing
