# Lab 03 — IPv4 Addressing, VLSM & Inter-Subnet Routing

## Objective

Design and implement a small-business network using Variable Length Subnet Masking (VLSM), static IPv4 addressing, default gateways, and inter-subnet routing.

The lab demonstrates efficient IPv4 address allocation, Cisco router configuration, and connectivity verification between multiple networks.

## Lab Environment

- **Simulator:** Cisco Packet Tracer
- **Router:** Cisco 2911 (R1)
- **Switches:** 3 Cisco 2960 switches
- **End devices:** 6 PCs
- **Parent network:** `192.168.10.0/24`
- **Routing method:** Directly connected networks

### Network Topology

The network consists of three departments:

- **Administration:** SW-ADMIN, ADMIN-PC1, ADMIN-PC2
- **IT:** SW-IT, IT-PC1, IT-PC2
- **Sales:** SW-SALES, SALES-PC1, SALES-PC2

Each departmental switch connects to a dedicated interface on R1, which provides routing between the three subnets.

The complete topology is saved in `03-IPv4-VLSM.pkt`.

## VLSM Addressing Plan

The original `192.168.10.0/24` network was divided into differently sized subnets according to each department's host requirements.

| Department | Required Hosts | Network | Subnet Mask | Usable Hosts |
|---|---:|---|---|---:|
| Administration | 50 | 192.168.10.0/26 | 255.255.255.192 | 62 |
| IT | 25 | 192.168.10.64/27 | 255.255.255.224 | 30 |
| Sales | 10 | 192.168.10.96/28 | 255.255.255.240 | 14 |

### Subnet Details

| Department | First Usable | Last Usable | Broadcast |
|---|---|---|---|
| Administration | 192.168.10.1 | 192.168.10.62 | 192.168.10.63 |
| IT | 192.168.10.65 | 192.168.10.94 | 192.168.10.95 |
| Sales | 192.168.10.97 | 192.168.10.110 | 192.168.10.111 |

The three subnets occupy 112 addresses in total, leaving 144 addresses unallocated for future use.

## Device Addressing

| Device | IPv4 Address | CIDR | Default Gateway |
|---|---|---|---|
| ADMIN-PC1 | 192.168.10.1 | /26 | 192.168.10.62 |
| ADMIN-PC2 | 192.168.10.2 | /26 | 192.168.10.62 |
| IT-PC1 | 192.168.10.65 | /27 | 192.168.10.94 |
| IT-PC2 | 192.168.10.66 | /27 | 192.168.10.94 |
| SALES-PC1 | 192.168.10.97 | /28 | 192.168.10.110 |
| SALES-PC2 | 192.168.10.98 | /28 | 192.168.10.110 |

Static IPv4 addresses, subnet masks, and default gateways were configured through the Packet Tracer PC interface.

## Router Configuration

### Interface Addressing

| Interface | Department | IPv4 Address | Subnet Mask |
|---|---|---|---|
| G0/0 | Administration | 192.168.10.62 | 255.255.255.192 |
| G0/1 | IT | 192.168.10.94 | 255.255.255.224 |
| G0/2 | Sales | 192.168.10.110 | 255.255.255.240 |

### Cisco IOS Configuration

```text
enable
configure terminal
hostname R1

interface gigabitEthernet 0/0
ip address 192.168.10.62 255.255.255.192
no shutdown

interface gigabitEthernet 0/1
ip address 192.168.10.94 255.255.255.224
no shutdown

interface gigabitEthernet 0/2
ip address 192.168.10.110 255.255.255.240
no shutdown

end
copy running-config startup-config
```

### Interface Verification

Command:

```text
show ip interface brief
```

Verified results:

| Interface | IP Address | Status | Protocol |
|---|---|---|---|
| GigabitEthernet0/0 | 192.168.10.62 | up | up |
| GigabitEthernet0/1 | 192.168.10.94 | up | up |
| GigabitEthernet0/2 | 192.168.10.110 | up | up |

## Routing Table Verification

Command:

```text
show ip route
```

R1 automatically installed three directly connected network routes and three local interface routes.

```text
C 192.168.10.0/26   is directly connected, GigabitEthernet0/0
L 192.168.10.62/32  is directly connected, GigabitEthernet0/0

C 192.168.10.64/27  is directly connected, GigabitEthernet0/1
L 192.168.10.94/32  is directly connected, GigabitEthernet0/1

C 192.168.10.96/28  is directly connected, GigabitEthernet0/2
L 192.168.10.110/32 is directly connected, GigabitEthernet0/2
```

- **C:** Directly connected network
- **L:** Local router interface address

No static routes were required because all three subnets were directly connected to R1.

The router had no gateway of last resort configured, which was acceptable for this isolated network.

## Connectivity Testing

### Within Each Department

Successful ICMP connectivity was verified between:

- ADMIN-PC1 and ADMIN-PC2
- IT-PC1 and IT-PC2
- SALES-PC1 and SALES-PC2

### Between Departments

Tests from ADMIN-PC1:

```text
ping 192.168.10.65
ping 192.168.10.97
```

Final results:

| Source | Destination | Successful Replies | Packet Loss |
|---|---|---|---|
| ADMIN-PC1 | IT-PC1 | 4/4 | 0% |
| ADMIN-PC1 | SALES-PC1 | 4/4 | 0% |

Initial tests experienced one lost packet per destination, consistent with ARP resolution during the first communication attempt.

Subsequent tests completed successfully with no packet loss.

## Traceroute Verification

Command:

```text
tracert 192.168.10.65
```

Observed path:

1. `192.168.10.62` — R1 Administration gateway
2. `192.168.10.65` — IT-PC1 destination

The traceroute confirmed that traffic between the two departments passed through one router.

## Packet Analysis

Packet Tracer Simulation Mode was used to inspect ARP and ICMP communication between different subnets.

Key observations:

- ADMIN-PC1 resolved the MAC address of its default gateway rather than the remote PC.
- The router forwarded the IP packet between directly connected networks.
- Ethernet source and destination MAC addresses changed when the router forwarded the packet.
- Source and destination IPv4 addresses remained unchanged.
- The router decremented the IPv4 TTL during forwarding.

The ICMP replies arrived with TTL 127, consistent with an initial TTL of 128 and one router hop.

## Key Findings

1. VLSM allows subnet sizes to match different host requirements.
2. A default gateway enables hosts to communicate with remote networks.
3. Routers automatically install routes for active, directly connected networks.
4. Router interfaces must have valid IPv4 configurations and be operational.
5. ARP resolves the MAC address of the local next hop.
6. Layer 2 Ethernet headers change across router hops, while ordinary routed IP addresses remain unchanged.
7. TTL limits packet forwarding and supports traceroute-based path discovery.
8. Connectivity should be verified using interface status, routing tables, ping, and traceroute.

## Conclusion

Successfully designed, configured, and verified a routed small-business network using VLSM.

All three departments were assigned appropriately sized subnets, connected through a Cisco 2911 router, and verified using Cisco IOS diagnostic commands and ICMP connectivity tests.
