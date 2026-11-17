# Lab 05 — OSPF Dynamic Routing

## Objective

Configure and verify single-area OSPF dynamic routing across three Cisco routers using Cisco Packet Tracer.

The goal is to establish OSPF neighbor adjacencies, automatically learn remote networks, and verify end-to-end connectivity without manually configured static routes.

## Lab Environment

- **Simulation software:** Cisco Packet Tracer
- **Routers:** 3 × Cisco 2911
- **Switches:** 3 × Cisco 2960
- **End devices:** 3 PCs
- **Routing protocol:** OSPFv2
- **OSPF process ID:** 1
- **OSPF area:** 0

## Network Topology

Three routers are connected in a linear topology:

`PC-R1 — SW-R1 — R1 — R2 — R3 — SW-R3 — PC-R3`

R2 also connects to SW-R2 and PC-R2.

### IP Addressing

| Device | Interface | IP Address | Subnet Mask |
|---|---|---|---|
| R1 | G0/0 | 192.168.1.1 | 255.255.255.0 |
| R1 | G0/1 | 10.0.12.1 | 255.255.255.252 |
| R2 | G0/0 | 192.168.2.1 | 255.255.255.0 |
| R2 | G0/1 | 10.0.12.2 | 255.255.255.252 |
| R2 | G0/2 | 10.0.23.1 | 255.255.255.252 |
| R3 | G0/0 | 192.168.3.1 | 255.255.255.0 |
| R3 | G0/2 | 10.0.23.2 | 255.255.255.252 |

### PC Configuration

| PC | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| PC-R1 | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| PC-R2 | 192.168.2.10 | 255.255.255.0 | 192.168.2.1 |
| PC-R3 | 192.168.3.10 | 255.255.255.0 | 192.168.3.1 |

## OSPF Configuration

### R1

```cisco
router ospf 1
 network 192.168.1.0 0.0.0.255 area 0
 network 10.0.12.0 0.0.0.3 area 0
```

### R2

```cisco
router ospf 1
 network 192.168.2.0 0.0.0.255 area 0
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.23.0 0.0.0.3 area 0
```

### R3

```cisco
router ospf 1
 network 192.168.3.0 0.0.0.255 area 0
 network 10.0.23.0 0.0.0.3 area 0
```

## Verification and Results

### OSPF Neighbors

The following commands were used:

```cisco
show ip protocols
show ip ospf neighbor
```

R2 established two OSPF adjacencies:

- R1 (`192.168.1.1`) — `FULL/DR`
- R3 (`192.168.3.1`) — `FULL/BDR`

R3 also confirmed its adjacency with R2 in the `FULL` state.

### Routing Table

The following command verified dynamically learned routes:

```cisco
show ip route
```

R1 automatically learned:

| Destination Network | Route Type | Next Hop | OSPF Cost |
|---|---|---|---|
| 10.0.23.0/30 | OSPF | 10.0.12.2 | 2 |
| 192.168.2.0/24 | OSPF | 10.0.12.2 | 2 |
| 192.168.3.0/24 | OSPF | 10.0.12.2 | 3 |

These routes appeared with the `O` routing-table code.

### End-to-End Connectivity

**PC-R1 to PC-R3**

```text
ping 192.168.3.10
```

Result: 3 of 4 replies received. The initial packet timed out, but subsequent requests succeeded.

**PC-R3 to PC-R1**

```text
ping 192.168.1.10
```

Result: 4 of 4 replies received, with 0% packet loss.

**Traceroute from PC-R1 to PC-R3**

```text
tracert 192.168.3.10
```

Observed path:

```text
1  192.168.1.1
2  10.0.12.2
3  10.0.23.2
4  192.168.3.10
```

The results confirmed successful communication across three routers.

## Key Concepts Learned

- Configuring OSPFv2 on Cisco routers
- Understanding OSPF process IDs and Area 0
- Using wildcard masks in OSPF network statements
- Establishing and verifying OSPF neighbor adjacencies
- Understanding FULL, DR, and BDR neighbor states
- Identifying OSPF-learned routes in the routing table
- Interpreting administrative distance and OSPF cost
- Verifying connectivity using ping and traceroute
- Saving Cisco IOS running configurations to startup configurations

## Conclusion

Successfully configured single-area OSPF across three routers, established neighbor adjacencies, and verified that remote networks were automatically learned through OSPF.

End-to-end connectivity was confirmed in both directions without manually configured static routes.

## Lab Files

- `OSPF-Dynamic-Routing.pkt` — Cisco Packet Tracer network topology and device configurations.
