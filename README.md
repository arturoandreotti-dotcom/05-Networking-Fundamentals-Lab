# Networking Fundamentals Lab

Hands-on networking project focused on designing, configuring, testing, and troubleshooting simulated enterprise-style networks using Cisco Packet Tracer.

This repository is **Project 05 of my 10-project IT Infrastructure & Cloud portfolio**, documenting my progression toward IT infrastructure, system administration, and cloud engineering.

## Project Overview

The objective of this project is to develop practical networking administration skills through progressively more complex lab environments.

Starting with basic LAN connectivity, the project advances through IPv4 addressing, routing, network services, VLAN segmentation, and structured troubleshooting.

Each lab includes hands-on configuration, verification, technical documentation, and troubleshooting observations.

## Technologies and Tools

- **Cisco Packet Tracer** — Network simulation and device configuration
- **Cisco IOS** — Router and switch administration
- **Ethernet and IPv4** — LAN connectivity and addressing
- **Static Routing and OSPF** — Layer 3 routing
- **VLANs and IEEE 802.1Q** — Network segmentation and trunking
- **DHCP and DNS** — Network infrastructure services
- **Git and GitHub** — Version control and documentation workflows
- **Visual Studio Code** — Repository and Markdown editing

## Network Labs

| Lab | Name | Topics Covered | Status |
|---|---|---|---|
| 01 | First LAN | Ethernet, switches, IPv4, ARP, ICMP, and connectivity testing | Completed |
| 02 | Network Troubleshooting and Cisco IOS | Interface diagnostics, connectivity troubleshooting, and Cisco CLI fundamentals | Completed |
| 03 | IPv4 Subnetting and Routing | IPv4 subnetting, default gateways, and routing between networks | Completed |
| 04 | Static Routing | Manually configured routes, routing tables, and multi-router connectivity | Completed |
| 05 | OSPF Dynamic Routing | OSPF Area 0, neighbor adjacencies, and dynamically learned routes | Completed |
| 06 | DHCP and DNS Services | DHCP address allocation, DNS resolution, and internal HTTP services | Completed |
| 07 | VLANs, Trunking and Inter-VLAN Routing | VLAN segmentation, 802.1Q trunks, and router-on-a-stick | Completed |
| 08 | Network Troubleshooting and Fault Diagnosis | Systematic troubleshooting of switching, routing, DHCP relay, and DNS failures | Completed |

An initial setup stage was also used to establish the repository structure, tooling, and Git workflow.

## Key Practical Skills

### Switching and LAN Administration

- Configured Cisco switches and Ethernet connections.
- Created and managed VLANs.
- Assigned access ports to appropriate VLANs.
- Configured and verified IEEE 802.1Q trunks.
- Investigated Spanning Tree forwarding states and trunk inconsistencies.

### IPv4 Addressing and Routing

- Configured IPv4 addresses and default gateways.
- Practiced subnetting and network addressing.
- Implemented static routes across multiple routers.
- Configured OSPF dynamic routing and verified neighbor relationships.
- Implemented inter-VLAN routing using router subinterfaces.
- Examined connected, local, static, and OSPF-learned routes.

### Network Infrastructure Services

- Configured DHCP address pools and automatic IP allocation.
- Configured DNS name resolution and internal HTTP services.
- Implemented DHCP relay across VLAN boundaries.
- Verified client configuration and server accessibility.

### Troubleshooting and Verification

- Diagnosed network connectivity problems using Cisco IOS.
- Investigated incorrect VLAN assignments and trunk restrictions.
- Identified missing DHCP relay configurations.
- Diagnosed incorrect DNS records.
- Repaired router subinterface configurations.
- Verified solutions using ping, traceroute, interface inspection, and service testing.

## Featured Lab — Small Company Network Troubleshooting

Lab 08 combines the technologies introduced throughout the project into a simulated small-company network.

### Environment

- 1 Cisco 2911 router
- 3 Cisco 2960 switches
- 3 client PCs
- 1 server
- 3 VLANs with separate IPv4 subnets
- Centralized DHCP, DNS, and HTTP services

### Network Segmentation

| VLAN | Department | Network |
|---|---|---|
| 10 | Administration | 192.168.10.0/24 |
| 20 | Employees | 192.168.20.0/24 |
| 30 | IT Services | 192.168.30.0/24 |

### Troubleshooting Scenarios

Five deliberately introduced network failures were investigated and resolved:

| Incident | Problem | Resolution |
|---|---|---|
| INC-001 | Incorrect access VLAN | Restored the correct VLAN assignment |
| INC-002 | Missing DHCP relay | Restored the router's helper address |
| INC-003 | Incorrect trunk allowed-VLAN list | Restored VLAN forwarding across the trunk |
| INC-004 | Incorrect DNS A record | Corrected hostname-to-IP mapping |
| INC-005 | Router subinterface misconfiguration | Restored VLAN encapsulation and gateway addressing |

Each incident was investigated through a structured workflow:

**Identify symptoms → Gather evidence → Isolate the fault → Apply the fix → Verify connectivity → Document findings**

The lab includes a known-good network baseline and separate Packet Tracer files containing the repaired configurations for each incident.

See the [Lab 08 documentation](labs/08-network-troubleshooting/README.md) for the complete topology, configurations, troubleshooting reports, and verification results.

## Repository Structure

```text
05-Networking-Fundamentals-Lab/
│
├── README.md
│
└── labs/
    ├── 01-first-lan/
    ├── 02-arp-icmp-analysis/
    ├── 03-ipv4-subnetting/
    ├── 04-static-routing/
    ├── 05-ospf-dynamic-routing/
    ├── 06-dhcp-dns-services/
    ├── 07-vlan-trunking/
    └── 08-network-troubleshooting/
```

Each practical lab contains technical documentation and relevant Cisco Packet Tracer project files.

## Documentation and Workflow

The project follows a structured Git and GitHub development workflow:

1. Create a dedicated feature branch for each lab.
2. Build and configure the network environment.
3. Test connectivity and verify device configurations.
4. Troubleshoot unexpected behavior.
5. Document implementation steps and results.
6. Commit changes with descriptive messages.
7. Open and merge a pull request.
8. Synchronize and clean up local branches.

Technical documentation is maintained in Markdown, with additional networking study notes organized separately in Obsidian.

## Project Outcomes

Through this project, I gained hands-on experience with fundamental networking technologies and the process of administering and troubleshooting simulated network infrastructure.

The labs progressed from basic LAN connectivity to multi-router networking, dynamic routing, network services, VLAN segmentation, and integrated troubleshooting.

The final troubleshooting lab reinforced the importance of systematic diagnosis, configuration verification, root cause analysis, and documenting corrective actions.

## Project Status

**Completed — 8 of 8 practical labs**

This project forms part of my ongoing 10-project IT Infrastructure & Cloud portfolio, progressing from foundational IT administration toward enterprise infrastructure, cloud technologies, automation, and DevOps.
