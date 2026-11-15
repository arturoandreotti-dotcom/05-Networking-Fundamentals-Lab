
# Lab 01 — First Local Area Network

## Objective

Build and configure a basic Ethernet LAN using Cisco Packet Tracer, verify IPv4 connectivity, and investigate ARP, ICMP, and switch MAC address learning.

## Network Topology

Two PCs connected to a Cisco 2960 switch using copper straight-through cables.

| Device | Interface | Connected To |
|---|---|---|
| PC0 | FastEthernet0 | Switch0 Fa0/1 |
| PC1 | FastEthernet0 | Switch0 Fa0/2 |

## IP Addressing

| Device | IPv4 Address | Subnet Mask |
|---|---|---|
| PC0 | 192.168.1.10 | 255.255.255.0 |
| PC1 | 192.168.1.20 | 255.255.255.0 |

Network: `192.168.1.0/24`

No default gateway was required because both PCs belong to the same subnet.

## Implementation

1. Added two PCs and one Cisco 2960 switch.
2. Connected both PCs using copper straight-through cables.
3. Configured static IPv4 addresses.
4. Verified connectivity using ICMP ping.
5. Inspected the ARP caches on both PCs.
6. Observed ARP and ICMP traffic in Simulation Mode.
7. Inspected the switch MAC address table.

## Verification

### Connectivity Tests

| Test | Result |
|---|---|
| PC0 → PC1 | Passed |
| PC1 → PC0 | Passed |
| Packet loss | 0% |

### ARP Verification

Both PCs successfully resolved the other device's IPv4 address to its MAC address.

### Switch MAC Address Table

Command:

```text
show mac address-table
```

Observed entries:

| VLAN | MAC Address | Type | Port |
|---|---|---|---|
| 1 | 0009.7c2c.886b | DYNAMIC | Fa0/1 |
| 1 | 0002.16a0.bc8a | DYNAMIC | Fa0/2 |

## Findings

- Devices in the same subnet can communicate without a router.
- ARP resolves IPv4 addresses to MAC addresses.
- ARP Requests use Ethernet broadcast.
- Switches flood broadcasts within the VLAN.
- Switches learn source MAC addresses and associate them with incoming ports.
- Known unicast frames can be forwarded through the appropriate switch port.
- ICMP Echo Requests and Replies can verify IP connectivity.

## Lab Files

- `01-First-LAN.pkt` — Cisco Packet Tracer topology and configuration.

## Conclusion

Successfully built and tested a basic Ethernet LAN.

Verified bidirectional connectivity, inspected ARP resolution, observed broadcast behavior, and confirmed dynamic MAC address learning on the switch.
