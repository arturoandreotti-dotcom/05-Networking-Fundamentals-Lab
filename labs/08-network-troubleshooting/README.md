# Lab 08 — Network Troubleshooting and Fault Diagnosis

## Objective

Build a simulated small-company network and troubleshoot deliberately introduced connectivity and service failures using a structured diagnostic methodology.

The lab covers Cisco switching, VLANs, 802.1Q trunking, inter-VLAN routing, DHCP relay, DNS, and router subinterface configuration.

The objective is to demonstrate practical network troubleshooting skills by identifying symptoms, investigating configuration and connectivity, determining root causes, applying corrective actions, and verifying service restoration.

## Lab Environment

- **Simulation platform:** Cisco Packet Tracer
- **Router:** 1 × Cisco 2911 (R1)
- **Switches:** 3 × Cisco 2960 (SW1, SW2, SW3)
- **Client devices:** 3 × PC-PT
- **Server:** 1 × Server-PT
- **Technologies:** IPv4, VLANs, IEEE 802.1Q, router-on-a-stick, DHCP, DHCP relay, DNS, HTTP, ICMP

## Network Topology

```text
                         R1
                      Gi0/0
                         |
                    802.1Q trunk
                         |
                    Gi0/1 SW1
                    /        \
             Gi0/2 /          \ Fa0/24
                  /            \
             Gi0/1 SW2       Gi0/1 SW3
              /     \         /      \
            PC1     PC2    Server1   PC3
           VLAN10  VLAN20  VLAN30   VLAN20
```

### VLANs and Subnets

| VLAN | Name | Subnet | Default Gateway |
|---|---|---|---|
| 10 | ADMIN | 192.168.10.0/24 | 192.168.10.1 |
| 20 | EMPLOYEES | 192.168.20.0/24 | 192.168.20.1 |
| 30 | IT_SERVICES | 192.168.30.0/24 | 192.168.30.1 |

### Device Addressing

| Device | VLAN | IP Address | Configuration |
|---|---|---|---|
| PC1 | 10 | 192.168.10.100/24 | DHCP |
| PC2 | 20 | 192.168.20.100/24 | DHCP |
| PC3 | 20 | 192.168.20.101/24 | DHCP |
| Server1 | 30 | 192.168.30.10/24 | Static |

### Physical Connections

| Source | Source Interface | Destination | Destination Interface |
|---|---|---|---|
| R1 | Gi0/0 | SW1 | Gi0/1 |
| SW1 | Gi0/2 | SW2 | Gi0/1 |
| SW1 | Fa0/24 | SW3 | Gi0/1 |
| SW2 | Fa0/1 | PC1 | Fa0 |
| SW2 | Fa0/2 | PC2 | Fa0 |
| SW3 | Fa0/1 | Server1 | Fa0 |
| SW3 | Fa0/2 | PC3 | Fa0 |

## Healthy Network Baseline

Before introducing failures, a fully operational network was configured and tested.

### Switching

Created VLANs 10, 20, and 30 across all three switches.

Configured access ports for the PCs and server and 802.1Q trunks between the switches and router.

Verified the configurations using:

```cisco
show vlan brief
show interfaces trunk
```

### Inter-VLAN Routing

Configured R1 using router-on-a-stick.

```cisco
interface GigabitEthernet0/0
 no shutdown

interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0

interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
```

All three subinterfaces were verified as `up/up`.

R1 automatically learned the directly connected networks.

### Centralized DHCP and Relay

Configured Server1 at `192.168.30.10` to provide DHCP addresses for VLANs 10 and 20.

| Pool | Address Range | Gateway | DNS |
|---|---|---|---|
| ADMIN | 192.168.10.100–149 | 192.168.10.1 | 192.168.30.10 |
| EMPLOYEES | 192.168.20.100–149 | 192.168.20.1 | 192.168.30.10 |

Configured DHCP relay on R1:

```cisco
interface GigabitEthernet0/0.10
 ip helper-address 192.168.30.10

interface GigabitEthernet0/0.20
 ip helper-address 192.168.30.10
```

Verified that all three PCs received addresses from the appropriate DHCP pools.

### DNS and HTTP

Configured Server1 as the DNS server and internal HTTP server.

Created an A record:

```text
intranet.company.test -> 192.168.30.10
```

Verified hostname resolution with `ping` and `nslookup`, and confirmed successful website access using:

```text
http://intranet.company.test
```

### Baseline Verification

| Test | Result |
|---|---|
| VLAN configurations | Passed |
| 802.1Q trunk forwarding | Passed |
| Router subinterfaces | Up/Up |
| DHCP relay | Passed |
| DHCP address allocation | Passed |
| Inter-VLAN connectivity | Passed |
| DNS name resolution | Passed |
| HTTP intranet access | Passed |

Saved the fully operational topology as `08-Healthy-Baseline.pkt` before introducing faults.

---

## Troubleshooting Methodology

Used the following troubleshooting workflow for each incident:

1. **Identify symptoms:** Determine which devices, services, and networks are affected.
2. **Gather evidence:** Inspect IP configuration, interface status, connectivity, and device configurations.
3. **Isolate the problem:** Narrow the suspected failure to a device, interface, VLAN, routing component, or service.
4. **Identify root cause:** Compare actual behavior against the expected configuration.
5. **Implement correction:** Apply the smallest appropriate configuration change.
6. **Verify resolution:** Repeat the original failing tests and confirm service restoration.
7. **Document findings:** Record symptoms, root cause, corrective action, and results.

Each incident was simulated using a separate Packet Tracer file derived from the healthy baseline.

---

## Incident 01 — Incorrect Access VLAN

**Ticket:** INC-001

**Reported issue:** PC2 could not access its default gateway or the company intranet, while another employee remained connected.

### Symptoms

PC2 retained a valid IPv4 configuration:

```text
IPv4 Address:     192.168.20.100
Default Gateway:  192.168.20.1
DNS Server:       192.168.30.10
```

Initial connectivity tests:

```text
ping 192.168.20.1   -> 0/4 replies
ping 192.168.30.10  -> 0/4 replies
```

### Investigation

Inspected VLAN assignments on SW2:

```cisco
show vlan brief
```

Discovered that `Fa0/2`, connected to PC2, belonged to VLAN 10 instead of VLAN 20.

### Root Cause

Incorrect access VLAN assignment on SW2's `Fa0/2` port.

The PC retained an IP address from `192.168.20.0/24`, but its Ethernet traffic entered VLAN 10.

### Resolution

```cisco
configure terminal
interface FastEthernet0/2
 switchport access vlan 20
end
```

### Verification

| Test | After Resolution |
|---|---|
| Gateway ping | 3/4 replies |
| Server ping | 4/4 replies |
| Intranet hostname ping | 4/4 replies |

**Outcome:** Resolved.

**Lesson:** A valid IP address does not guarantee that the client is connected to the correct VLAN.

---

## Incident 02 — Missing DHCP Relay

**Ticket:** INC-002

**Reported issue:** Employee computers could not obtain IPv4 addresses after restarting or renewing DHCP.

### Symptoms

After releasing the existing address:

```cmd
ipconfig /release
ipconfig /renew
```

PC2 reported:

```text
DHCP request failed.
IPv4 Address:     0.0.0.0
Default Gateway:  0.0.0.0
```

### Investigation

Compared the configurations of R1's VLAN 10 and VLAN 20 subinterfaces using:

```cisco
show running-config
```

Found that the DHCP relay configuration was missing from `Gi0/0.20`.

### Root Cause

Missing `ip helper-address` on the interface receiving DHCP broadcasts from VLAN 20.

Because Server1 belonged to VLAN 30, the initial DHCP broadcasts from VLAN 20 could not reach the DHCP server without a relay agent.

### Resolution

```cisco
configure terminal
interface GigabitEthernet0/0.20
 ip helper-address 192.168.30.10
end
```

### Verification

PC2 successfully renewed its DHCP configuration:

```text
IPv4 Address:     192.168.20.100
Subnet Mask:      255.255.255.0
Default Gateway:  192.168.20.1
DNS Server:       192.168.30.10
```

The company intranet hostname was also reachable.

**Outcome:** Resolved.

**Lesson:** Normal routed IP traffic and initial DHCP broadcast traffic have different forwarding requirements.

---

## Incident 03 — Incorrect Trunk Allowed-VLAN List

**Ticket:** INC-003

**Reported issue:** PC3 lost connectivity to its default gateway and company services.

### Symptoms

PC3 retained the correct IPv4 configuration but could not reach either destination:

```text
ping 192.168.20.1   -> 0/4 replies
ping 192.168.30.10  -> 0/4 replies
```

### Investigation

Inspected SW3's uplink:

```cisco
show interfaces trunk
```

Found:

```text
Vlans allowed on trunk
Gig0/1      10,30
```

VLAN 20 was missing.

Also investigated STP information using:

```cisco
show spanning-tree vlan 10
```

Confirmed that VLAN 10's Gi0/1 port was in the designated forwarding state during the check.

### Root Cause

VLAN 20 was excluded from the allowed-VLAN list on SW3's `Gi0/1` trunk.

PC3 could not reach the router because its VLAN traffic was not permitted across SW3's only uplink.

### Resolution

```cisco
configure terminal
interface GigabitEthernet0/1
 switchport trunk allowed vlan add 20
end
```

### Verification

The trunk showed VLANs 10, 20, and 30 as allowed and forwarding.

| Test | After Resolution |
|---|---|
| Gateway ping | 4/4 replies |
| Server ping | 4/4 replies |
| Intranet hostname ping | 4/4 replies |

**Outcome:** Resolved.

**Lesson:** A trunk can be operational while still excluding a VLAN required by connected devices.

---

## Incident 04 — Incorrect DNS A Record

**Ticket:** INC-004

**Reported issue:** Users could reach the intranet server by IP address but not by hostname.

### Symptoms

Direct IP connectivity:

```text
ping 192.168.30.10 -> 3/4 replies
```

Hostname connectivity:

```text
ping intranet.company.test -> 0/4 replies
```

The hostname resolved to the wrong IP address.

### Investigation

Executed:

```cmd
nslookup intranet.company.test
```

Observed:

```text
DNS Server: 192.168.30.10
Name: intranet.company.test
Address: 192.168.30.99
```

### Root Cause

The DNS A record contained `192.168.30.99` instead of the server's actual IP address, `192.168.30.10`.

### Resolution

Corrected the A record on Server1:

```text
intranet.company.test -> 192.168.30.10
```

### Verification

- `nslookup` returned `192.168.30.10`.
- Ping by hostname returned 4/4 replies.
- The intranet website loaded successfully in the PC browser.

**Outcome:** Resolved.

**Lesson:** DNS can respond successfully while providing an incorrect address. DNS data accuracy must be verified separately from IP connectivity.

---

## Incident 05 — Router Subinterface Configuration Failure

**Ticket:** INC-005

**Reported issue:** The Administration department could not access its gateway or company services.

### Symptoms

PC1 retained:

```text
IPv4 Address:     192.168.10.100
Default Gateway:  192.168.10.1
```

But both tests failed:

```text
ping 192.168.10.1   -> 0/4 replies
ping 192.168.30.10  -> 0/4 replies
```

### Investigation

Inspected R1 with:

```cisco
show running-config
show ip interface brief
```

Discovered that the VLAN 10 subinterface was missing both its 802.1Q encapsulation and its IPv4 address.

During the repair, restoring the encapsulation alone was insufficient because the subinterface still displayed `no ip address`.

### Root Cause

Removal of `encapsulation dot1Q 10` also caused the subinterface's IPv4 configuration to be removed in the simulated environment.

Without the VLAN association and gateway address, R1 could not provide the intended Layer 3 service for VLAN 10.

### Resolution

```cisco
configure terminal
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
end
```

### Verification

`show ip interface brief` confirmed that `Gi0/0.10` was `up/up` with IPv4 address `192.168.10.1`.

| Test | After Resolution |
|---|---|
| Gateway ping | 4/4 replies |
| Server ping | 4/4 replies |
| Intranet hostname ping | 4/4 replies |

**Outcome:** Resolved.

**Lesson:** Correcting one missing configuration command may not restore other settings that were removed or affected. Always verify the complete interface configuration.

---

## Troubleshooting Commands Used

### Cisco IOS

| Command | Purpose |
|---|---|
| `show vlan brief` | Verify VLAN membership and access ports |
| `show interfaces trunk` | Inspect trunk status, allowed VLANs, and forwarding VLANs |
| `show spanning-tree vlan 10` | Inspect STP state for VLAN 10 |
| `show ip interface brief` | Check router interface addressing and operational state |
| `show ip route` | Inspect router reachability information |
| `show running-config` | Inspect active device configuration |
| `copy running-config startup-config` | Save device configuration |

### Packet Tracer PC Commands

| Command | Purpose |
|---|---|
| `ipconfig /all` | View IPv4, gateway, DHCP, and DNS information |
| `ipconfig /release` | Release DHCP-assigned IPv4 configuration |
| `ipconfig /renew` | Request DHCP configuration |
| `ping <address>` | Test IP connectivity |
| `tracert <address>` | Inspect the Layer 3 route |
| `nslookup <hostname>` | Query DNS resolution |

## Final Incident Summary

| Incident | Root Cause | Resolution |
|---|---|---|
| INC-001 | Incorrect access VLAN | Restored VLAN 20 on SW2 Fa0/2 |
| INC-002 | Missing DHCP relay | Restored helper address on R1 Gi0/0.20 |
| INC-003 | VLAN 20 excluded from trunk | Added VLAN 20 to SW3 Gi0/1 allowed list |
| INC-004 | Incorrect DNS A record | Updated hostname mapping to 192.168.30.10 |
| INC-005 | Missing encapsulation and gateway IP | Restored dot1Q encapsulation and IPv4 address |

**Results: 5 incidents investigated, repaired, and verified.**

## Key Skills Demonstrated

- Structured network troubleshooting
- Cisco IOS command-line administration
- VLAN configuration and fault diagnosis
- 802.1Q trunk verification
- Spanning Tree verification
- Router-on-a-stick configuration
- DHCP relay troubleshooting
- DNS record analysis and correction
- Layer 2 versus Layer 3 fault isolation
- Service restoration and verification
- Technical incident documentation

## Lab Files

| File | Description |
|---|---|
| `08-Healthy-Baseline.pkt` | Fully operational reference topology |
| `08-Incident-01.pkt` | Incorrect access VLAN scenario, repaired |
| `08-Incident-02.pkt` | Missing DHCP relay scenario, repaired |
| `08-Incident-03.pkt` | Trunk allowed-VLAN scenario, repaired |
| `08-Incident-04.pkt` | Incorrect DNS record scenario, repaired |
| `08-Incident-05.pkt` | Router subinterface failure scenario, repaired |

## Conclusion

Successfully built and validated a simulated small-company network with three VLANs, inter-VLAN routing, centralized DHCP services, DHCP relay, DNS, and an internal HTTP server.

Performed five fault-diagnosis exercises covering switching, routing, address allocation, and name resolution.

Each incident was approached using a repeatable troubleshooting process, and every repair was validated with relevant Cisco IOS and network connectivity tests.

The lab demonstrates foundational network administration and troubleshooting skills relevant to IT support, junior network administration, and systems administration roles.
