# Lab 06 — DHCP and DNS Services

## Objective

Configure and verify DHCP, DNS, and HTTP services in a small office network using Cisco Packet Tracer.

The goal is to automatically assign IP configurations to client devices, resolve hostnames to IP addresses, and access an internal web server using a domain name.

## Lab Environment

- **Simulation software:** Cisco Packet Tracer
- **Router:** 1 × Cisco 2911
- **Switch:** 1 × Cisco 2960
- **Server:** 1 × Server-PT
- **Clients:** 3 × PC-PT
- **Network:** `192.168.10.0/24`

### Network Services

| Device | Role |
|---|---|
| R1 | DHCP server and default gateway |
| SW1 | LAN connectivity |
| Server1 | DNS and HTTP server |
| PC1, PC2, PC3 | DHCP clients |

## Network Topology

```text
                    R1
            DHCP / Gateway
              192.168.10.1
                    |
                   SW1
          __________|__________
         |       |      |      |
        PC1     PC2    PC3   Server1
                               |
                         DNS / HTTP
                         192.168.10.2
```

All devices connect to SW1 using copper straight-through cables.

## IP Addressing

| Device | IPv4 Address | Assignment |
|---|---|---|
| R1 G0/0 | 192.168.10.1/24 | Static |
| Server1 | 192.168.10.2/24 | Static |
| PC1 | 192.168.10.100/24 | DHCP |
| PC2 | 192.168.10.101/24 | DHCP |
| PC3 | 192.168.10.102/24 | DHCP |

- **Default gateway:** `192.168.10.1`
- **DNS server:** `192.168.10.2`

## Implementation

### 1. Router Interface Configuration

Configured R1's LAN interface:

```cisco
enable
configure terminal
hostname R1

interface GigabitEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown

end
```

Verified the interface using:

```cisco
show ip interface brief
```

The interface reported an operational status of `up/up`.

### 2. DHCP Server Configuration

Configured R1 to provide automatic IPv4 configurations to LAN clients.

```cisco
configure terminal

ip dhcp excluded-address 192.168.10.1 192.168.10.99

ip dhcp pool OFFICE-LAN
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 192.168.10.2

end
```

The exclusion range reserves addresses `192.168.10.1–192.168.10.99` from DHCP allocation.

This protects infrastructure addresses, including the default gateway and DNS server.

Client addresses are dynamically assigned from the remaining available addresses in the subnet.

Verified the configuration with:

```cisco
show running-config | section dhcp
```

### 3. DNS Server Configuration

Configured Server1 with a static IPv4 address:

```text
IP Address:       192.168.10.2
Subnet Mask:      255.255.255.0
Default Gateway:  192.168.10.1
DNS Server:       192.168.10.2
```

Enabled DNS under **Services → DNS** and created the following resource record:

| Record Type | Hostname | IPv4 Address |
|---|---|---|
| A | intranet.company.test | 192.168.10.2 |

This allows clients to resolve the intranet hostname to Server1's IPv4 address.

### 4. HTTP Server Configuration

Enabled HTTP on Server1 through **Services → HTTP**.

The default Cisco Packet Tracer webpage was used to verify web service accessibility.

## Verification and Results

### DHCP Client Configuration

Configured PC1, PC2, and PC3 to obtain their IPv4 configurations automatically using DHCP.

Observed results:

| Client | Assigned IPv4 Address | Result |
|---|---|---|
| PC1 | 192.168.10.100 | Successful |
| PC2 | 192.168.10.101 | Successful |
| PC3 | 192.168.10.102 | Successful |

On PC1, the following command was used to renew the DHCP lease:

```cmd
ipconfig /renew
```

The resulting configuration included:

```text
IP Address:       192.168.10.100
Subnet Mask:      255.255.255.0
Default Gateway:  192.168.10.1
DNS Server:       192.168.10.2
```

### DNS Resolution

Executed the following command on PC1:

```cmd
nslookup intranet.company.test
```

Result:

```text
Server:  192.168.10.2
Address: 192.168.10.2

Non-authoritative answer:
Name:    intranet.company.test
Address: 192.168.10.2
```

DNS successfully resolved the hostname to Server1's address.

### HTTP Connectivity

Opened the following URL using the web browser on PC2:

```text
http://intranet.company.test
```

The Cisco Packet Tracer webpage loaded successfully.

This confirmed that a DHCP client could use its configured DNS server to resolve the hostname and access the HTTP service.

## Troubleshooting — Incorrect DHCP Configuration

### Problem

Initially, PC1 obtained an unexpected DHCP configuration:

```text
IP Address:       192.168.10.3
Subnet Mask:      255.255.255.0
Default Gateway:  0.0.0.0
DNS Server:       192.168.10.2
```

The address did not match the intended DHCP allocation range, and the default gateway was missing.

### Investigation

The following troubleshooting steps were performed:

1. Verified basic IP connectivity between Server1 and R1.
2. Checked whether Server1's DHCP service was enabled.
3. Inspected the default `serverPool` and custom `OFFICE-LAN` pools.
4. Identified overlapping DHCP pool configurations.
5. Used `ipconfig /release` and `ipconfig /renew` to test client address assignment.
6. Inspected DHCP configuration commands on Cisco IOS.

### Resolution

Disabled DHCP on Server1 and configured R1 as the DHCP server.

Server1 continued providing DNS and HTTP services.

After renewing PC1's DHCP lease, the client received:

```text
IP Address:       192.168.10.100
Default Gateway:  192.168.10.1
DNS Server:       192.168.10.2
```

All three PCs subsequently received valid, distinct DHCP addresses.

### Lessons Learned

- A DHCP service must be enabled to respond to client requests.
- Incorrect or overlapping DHCP pools can cause unexpected client configurations.
- DHCP troubleshooting should include checking the assigned IP address, subnet mask, gateway, and DNS settings.
- Cisco IOS routers can provide DHCP services directly.
- DHCP and DNS are separate services and can run on different devices.
- DHCP clients can renew their configuration without manually assigning IP addresses.

## Key Concepts Learned

- Static versus dynamic IP addressing
- DHCP clients and servers
- DHCP address pools and exclusions
- DHCP lease renewal
- Default gateway and DNS configuration through DHCP
- DNS A records and hostname resolution
- HTTP service verification
- Cisco IOS DHCP configuration and troubleshooting

## Conclusion

Successfully configured a small office network with router-based DHCP and server-based DNS and HTTP services.

All three clients automatically received unique IPv4 configurations, DNS successfully resolved the internal hostname, and the intranet webpage was accessible from a DHCP client.

The lab also demonstrated systematic troubleshooting of incorrect DHCP configuration and overlapping address pools.

## Lab Files

- `06-DHCP-DNS-Services.pkt` — Cisco Packet Tracer topology and device configurations.
