# Lab 07 — VLANs, Trunking and Inter-VLAN Routing

## Objective

Configure and verify VLAN segmentation, IEEE 802.1Q trunking, and inter-VLAN routing using Cisco Packet Tracer.

The lab demonstrates how multiple VLANs can share physical switching infrastructure while remaining logically separated, and how a router can provide communication between them using router-on-a-stick.

## Lab Environment

- **Simulation software:** Cisco Packet Tracer
- **Router:** 1 × Cisco 2911
- **Switches:** 2 × Cisco 2960
- **End devices:** 4 × PC-PT
- **Technologies:** VLANs, IEEE 802.1Q, router-on-a-stick, IPv4 routing

## Network Topology

```text
                    R1
                  G0/0
                    |
                802.1Q trunk
                    |
                   SW1
                 /  |  \
               PC1 PC2  Trunk
                        |
                       SW2
                      /   \
                    PC3   PC4
```

### VLAN Configuration

| VLAN ID | Name | Subnet |
|---|---|---|
| 10 | ADMIN | 192.168.10.0/24 |
| 20 | EMPLOYEES | 192.168.20.0/24 |

### PC Addressing

| Device | Switch | VLAN | IP Address | Default Gateway |
|---|---|---|---|---|
| PC1 | SW1 | 10 | 192.168.10.10/24 | 192.168.10.1 |
| PC2 | SW1 | 20 | 192.168.20.20/24 | 192.168.20.1 |
| PC3 | SW2 | 10 | 192.168.10.30/24 | 192.168.10.1 |
| PC4 | SW2 | 20 | 192.168.20.40/24 | 192.168.20.1 |

### Physical Connections

| Connection | Interfaces | Cable |
|---|---|---|
| PC1 — SW1 | Fa0 — Fa0/1 | Straight-through |
| PC2 — SW1 | Fa0 — Fa0/2 | Straight-through |
| PC3 — SW2 | Fa0 — Fa0/1 | Straight-through |
| PC4 — SW2 | Fa0 — Fa0/2 | Straight-through |
| SW1 — SW2 | Gi0/1 — Gi0/1 | Crossover |
| R1 — SW1 | Gi0/0 — Gi0/2 | Straight-through |

## Implementation

### 1. VLAN Creation

Created VLAN 10 and VLAN 20 on both switches.

```cisco
configure terminal

vlan 10
 name ADMIN

vlan 20
 name EMPLOYEES

end
```

Verified using:

```cisco
show vlan brief
```

### 2. Access Port Configuration

Configured PC-facing interfaces on SW1 and SW2.

```cisco
configure terminal

interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 10

interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 20

end
```

Each access port was assigned to its corresponding VLAN.

### 3. Switch-to-Switch Trunk

Configured `Gi0/1` on both SW1 and SW2 as an IEEE 802.1Q trunk.

```cisco
configure terminal

interface gigabitEthernet 0/1
 switchport mode trunk

end
```

Verified using:

```cisco
show interfaces trunk
```

The trunk successfully carried VLAN 10 and VLAN 20 traffic between the switches.

### 4. Router-on-a-Stick Configuration

Enabled R1's physical interface and created separate logical subinterfaces for each VLAN.

```cisco
configure terminal

interface gigabitEthernet 0/0
 no shutdown

interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface gigabitEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0

end
```

The physical interface `G0/0` remained unnumbered.

Each subinterface provided a default gateway for its VLAN.

### 5. Router-Facing Trunk

Configured SW1's `Gi0/2`, connected to R1, as a trunk.

```cisco
configure terminal

interface gigabitEthernet 0/2
 switchport mode trunk

end
```

Verified that both SW1 trunk ports were operational and forwarding VLAN 10 and VLAN 20 traffic.

## Verification and Results

### VLAN Assignment

Both switches reported:

| VLAN | Name | Access Port |
|---|---|---|
| 10 | ADMIN | Fa0/1 |
| 20 | EMPLOYEES | Fa0/2 |

### Same-VLAN Connectivity

**PC1 → PC3 (VLAN 10)**

```cmd
ping 192.168.10.30
```

Result: 4/4 replies received, 0% packet loss.

**PC2 → PC4 (VLAN 20)**

```cmd
ping 192.168.20.40
```

Result: 4/4 replies received, 0% packet loss.

These tests confirmed that VLAN traffic crossed the switch-to-switch trunk successfully.

### VLAN Isolation

Before configuring inter-VLAN routing, PC1 attempted to communicate with PC2:

```cmd
ping 192.168.20.20
```

Result: 0/4 replies received.

This demonstrated that devices in different VLANs could not communicate without Layer 3 routing.

### Inter-VLAN Routing

After configuring router-on-a-stick and the PCs' default gateways:

**PC1 → PC2**

```cmd
ping 192.168.20.20
```

Result: 3/4 replies received. Subsequent replies succeeded.

**PC1 → PC4**

```cmd
ping 192.168.20.40
```

Result: 3/4 replies received. Subsequent replies succeeded.

The initial timeouts were consistent with address-resolution behavior during initial communication.

### Traceroute

Executed on PC1:

```cmd
tracert 192.168.20.40
```

Observed path:

```text
1  192.168.10.1
2  192.168.20.40
```

This confirmed that traffic between VLANs passed through R1.

### Routing Table

Verified using:

```cisco
show ip route
```

R1 contained the following directly connected routes:

```text
C 192.168.10.0/24 is directly connected, GigabitEthernet0/0.10
L 192.168.10.1/32 is directly connected, GigabitEthernet0/0.10

C 192.168.20.0/24 is directly connected, GigabitEthernet0/0.20
L 192.168.20.1/32 is directly connected, GigabitEthernet0/0.20
```

No static routes or dynamic routing protocols were necessary because both VLAN subnets were directly connected to R1.

## Troubleshooting and Observations

### Cisco IOS Command Modes

Some commands initially failed because they were entered in the wrong configuration mode.

For example, `show interfaces trunk` was entered in interface configuration mode.

Resolved by returning to privileged EXEC mode with `end`, or by using the `do` prefix when appropriate.

### Spanning Tree Transition

During trunk configuration, interfaces temporarily changed state.

A temporary VLAN inconsistency was also observed when one end of the switch-to-switch connection was configured as a trunk and the other was not.

After configuring both ends consistently, the trunks reported VLANs 1, 10, and 20 in the forwarding state.

### Verification

Repeated `show interfaces trunk` after configuration changes to confirm the final operational state rather than relying only on command acceptance.

## Key Concepts Learned

- VLAN segmentation and broadcast domains
- Access ports and VLAN membership
- IEEE 802.1Q VLAN tagging
- Native VLAN behavior
- Switch-to-switch trunking
- Spanning Tree consistency and forwarding states
- Router-on-a-stick and logical subinterfaces
- Inter-VLAN routing using a single physical router interface
- Connected and local routes in Cisco IOS
- Connectivity verification with ping and traceroute

## Conclusion

Successfully configured and verified two VLANs distributed across two Cisco switches.

Same-VLAN communication was confirmed across the 802.1Q trunk, while traffic between different VLANs was initially isolated.

Router-on-a-stick was then implemented using two logical router subinterfaces, enabling successful inter-VLAN communication without static routes or a dynamic routing protocol.

All three Cisco device configurations were saved to startup-config.

## Lab Files

- `07-VLAN-Trunking.pkt` — Cisco Packet Tracer topology and device configurations.
