# Lab 04 — Static Routing Between Routers

## Objective

Configure and troubleshoot static IPv4 routing between two local area networks connected through separate Cisco routers.

The lab demonstrates routing tables, next-hop addresses, bidirectional routing, and verification using ICMP and traceroute.

## Lab Environment

- **Simulator:** Cisco Packet Tracer
- **Routers:** 2 Cisco 2911 (R1, R2)
- **Switches:** 2 Cisco 2960 (SW-LAN1, SW-LAN2)
- **End devices:** 2 PCs
- **Routing method:** Static IPv4 routing

The completed topology is saved as `04-Static-Routing.pkt`.

## Network Topology

```text
PC-LAN1 --- SW-LAN1 --- R1 --- R2 --- SW-LAN2 --- PC-LAN2
192.168.1.0/24          |      |           192.168.2.0/24
                       10.0.0.0/30
```

### Addressing Plan

| Device | Interface | IPv4 Address | Subnet Mask |
|---|---|---|---|
| PC-LAN1 | Fa0 | 192.168.1.10 | 255.255.255.0 |
| R1 | G0/0 | 192.168.1.1 | 255.255.255.0 |
| R1 | G0/1 | 10.0.0.1 | 255.255.255.252 |
| R2 | G0/1 | 10.0.0.2 | 255.255.255.252 |
| R2 | G0/0 | 192.168.2.1 | 255.255.255.0 |
| PC-LAN2 | Fa0 | 192.168.2.10 | 255.255.255.0 |

PC-LAN1 uses `192.168.1.1` as its default gateway.

PC-LAN2 uses `192.168.2.1` as its default gateway.

## Router Configuration

### R1

```text
enable
configure terminal
hostname R1

interface g0/0
ip address 192.168.1.1 255.255.255.0
no shutdown

interface g0/1
ip address 10.0.0.1 255.255.255.252
no shutdown

ip route 192.168.2.0 255.255.255.0 10.0.0.2

end
copy running-config startup-config
```

### R2

```text
enable
configure terminal
hostname R2

interface g0/0
ip address 192.168.2.1 255.255.255.0
no shutdown

interface g0/1
ip address 10.0.0.2 255.255.255.252
no shutdown

ip route 192.168.1.0 255.255.255.0 10.0.0.1

end
copy running-config startup-config
```

## Verification

### Interface Status

Command:

```text
show ip interface brief
```

Both configured Gigabit Ethernet interfaces on each router were verified as operational (`up/up`).

### Router-to-Router Connectivity

Commands:

```text
R1# ping 10.0.0.2
R2# ping 10.0.0.1
```

Observed results:

- R1 → R2: 4/5 successful replies initially.
- R2 → R1: 5/5 successful replies.

The initial lost packet was consistent with ARP resolution.

### Routing Tables

Command:

```text
show ip route
```

Verified on R1:

```text
S 192.168.2.0/24 [1/0] via 10.0.0.2
```

Verified on R2:

```text
S 192.168.1.0/24 [1/0] via 10.0.0.1
```

Each router maintained directly connected (`C`), local (`L`), and static (`S`) routes.

### End-to-End Connectivity

Commands:

On PC-LAN1:

```text
ping 192.168.2.10
```

On PC-LAN2:

```text
ping 192.168.1.10
```

**Result:** Communication succeeded in both directions after configuring the static routes.

Successful replies showed TTL 126, consistent with packets passing through two routers when originating with TTL 128.

### Traceroute

Command:

```text
tracert 192.168.2.10
```

Observed path:

| Hop | Address | Device |
|---|---|---|
| 1 | 192.168.1.1 | R1 |
| 2 | 10.0.0.2 | R2 |
| 3 | 192.168.2.10 | PC-LAN2 |

The traceroute verified that traffic crossed both routers before reaching its destination.

## Troubleshooting Exercise

### Scenario — Missing Static Route

Removed R1's route:

```text
configure terminal
no ip route 192.168.2.0 255.255.255.0 10.0.0.2
end
```

**Problem:** R1 no longer had a route to LAN 2.

Even though the physical router interfaces remained operational, R1 could not forward traffic to the remote LAN.

### Resolution

Restored the static route:

```text
configure terminal
ip route 192.168.2.0 255.255.255.0 10.0.0.2
end
```

Verified that the static route reappeared in the routing table and connectivity was restored.

## Configuration Persistence

Saved both routers using:

```text
copy running-config startup-config
```

Saved the Packet Tracer topology, closed the project, and reopened it.

Verified that the static routes were still present on both R1 and R2.

## Key Findings

1. Directly connected networks are automatically added to a router's routing table.
2. Remote networks require an appropriate route, which can be configured statically or learned dynamically.
3. Static routes specify a destination network, subnet mask, and next hop or outgoing interface.
4. The destination mask determines which IP addresses match a route.
5. Bidirectional communication requires valid forwarding and return paths.
6. A working router-to-router link does not guarantee reachability of remote LANs.
7. `show ip route`, `ping`, and `tracert` are useful for verifying network routing.
8. Saving and reopening network configurations helps verify persistence.

## Conclusion

Successfully designed, configured, tested, and troubleshot a two-router IPv4 network using static routing.

Both remote LANs were reachable, routing tables contained the expected entries, and the final configurations were preserved in the saved Packet Tracer project.
