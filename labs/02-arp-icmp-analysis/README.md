# Lab 02 — ARP, ICMP & Network Troubleshooting

## Objective

Investigate network connectivity failures using Cisco Packet Tracer, ARP, ICMP, and Cisco IOS diagnostic commands.

The lab focuses on identifying problems at different OSI layers and restoring normal connectivity.

## Lab Environment

- **Simulator:** Cisco Packet Tracer
- **Switch:** Cisco Catalyst 2960
- **End devices:** 2 PCs
- **Network:** 192.168.1.0/24
- **VLAN:** 1

### Network Configuration

| Device | IPv4 Address | Subnet Mask | Switch Port |
|---|---|---|---|
| PC0 | 192.168.1.10 | 255.255.255.0 | Fa0/1 |
| PC1 | 192.168.1.20 | 255.255.255.0 | Fa0/2 |

No default gateway was configured because the original topology contains only one subnet.

## Troubleshooting Scenarios

### Scenario 1 — Different IPv4 Subnets

**Modification:** Changed PC1's IP address to `192.168.2.20/24`.

**Test:**

```text
ping 192.168.2.20
arp -a
```

**Observations:**

- Ping failed with 100% packet loss.
- PC0's ARP table contained no entries.
- No ARP or ICMP packets appeared in Simulation Mode.

**Explanation:**

PC0 recognized that the destination belonged to a different subnet. Without a default gateway or another suitable route, PC0 could not forward traffic to that network.

### Scenario 2 — Nonexistent Local Host

**Test:**

```text
ping 192.168.1.99
```

**Observations:**

- PC0 generated ARP Requests.
- The switch flooded the requests to other active ports in the VLAN.
- No device responded with an ARP Reply.
- The ping failed.

**Explanation:**

The destination was within the local subnet, but no device owned the requested IPv4 address.

Without resolving the destination's MAC address, PC0 could not send the ICMP Echo Request to that host.

### Scenario 3 — Disconnected Ethernet Cable

**Modification:** Disconnected PC1 from switch port Fa0/2.

**Observations:**

- PC0 generated ARP Requests.
- The switch received the broadcasts.
- PC1 could not receive the requests because its physical connection was disconnected.
- No ARP Reply was generated.

**Explanation:**

A Layer 1 physical connectivity failure prevented communication.

### Scenario 4 — Administratively Disabled Switch Port

**Configuration:**

```text
enable
configure terminal
interface fastEthernet 0/2
shutdown
end
```

**Verification:**

```text
show interfaces fastEthernet 0/2
```

**Observed state:**

```text
FastEthernet0/2 is administratively down,
line protocol is down (disabled)
```

**Resolution:**

```text
configure terminal
interface fastEthernet 0/2
no shutdown
end
```

The interface returned to `up/up`.

## Switch Verification

### Interface Status

```text
show interfaces status
show interfaces fastEthernet 0/2
```

After restoring the interface, Fa0/2 reported:

- Interface up
- Line protocol up
- Full-duplex
- 100 Mbps
- No recorded input or CRC errors
- No recorded output drops

### MAC Address Table

```text
show mac address-table
```

| VLAN | MAC Address | Type | Port |
|---|---|---|---|
| 1 | 0002.16a0.bc8a | DYNAMIC | Fa0/2 |
| 1 | 0009.7c2c.886b | DYNAMIC | Fa0/1 |

The switch successfully learned the MAC addresses of both connected PCs.

### VLAN Verification

```text
show vlan brief
```

Both Fa0/1 and Fa0/2 belonged to VLAN 1, which was active.

## Final Verification

Restored the original network configuration:

- PC0: `192.168.1.10/24`
- PC1: `192.168.1.20/24`
- Both switch ports enabled
- Both physical links operational

Bidirectional connectivity was restored.

The switch configuration was saved using:

```text
copy running-config startup-config
```

The complete topology was saved as `02-ARP-ICMP-Analysis.pkt`.

## Key Findings

1. Physical connectivity does not guarantee IP connectivity.
2. Hosts use their subnet masks to determine whether destinations are local or remote.
3. ARP resolves IPv4 addresses to MAC addresses on the local network.
4. Switches flood ARP broadcasts within the relevant VLAN.
5. Cisco IOS interface states help distinguish physical failures from administrative shutdowns.
6. MAC address tables and VLAN information help verify Layer 2 connectivity.
7. Troubleshooting is more effective when performed systematically across OSI layers.

## Conclusion

Successfully investigated multiple network failures, identified their causes, restored connectivity, and verified the final configuration using Cisco IOS diagnostic commands.
