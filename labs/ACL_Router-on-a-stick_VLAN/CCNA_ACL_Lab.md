# CCNA Lab: VLANs, Router-on-a-Stick, and ACL Security

## Topology

Devices: - 1 Router (R1) - 1 Switch (SW1) - PC0 (HR) - PC1 (SALES) -
Server0 (SERVERS)

Connections: - PC0 → SW1 Fa0/1 - PC1 → SW1 Fa0/2 - Server0 → SW1 Fa0/3 -
SW1 Fa0/24 → R1 G0/0

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
