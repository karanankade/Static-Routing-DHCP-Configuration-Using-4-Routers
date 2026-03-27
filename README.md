# Lab Report: Static Routing & DHCP Configuration Using 4 Routers

## 1. Objective
To configure static routing between four Cisco routers using a subnetted Class B network (180.180.0.0/16 divided into /19 subnets), implement DHCP to automatically assign IP addresses to LAN endpoints, and verify end‑to‑end connectivity across the network.

## 2. Topology Description
The network physical layout follows a linear topology (Router0 — Router1 — Router2 — Router3) consisting of:
- **4 Routers**: Router0, Router1, Router2, Router3
- **4 Switches**: Connecting individual LAN endpoints to their respective routers
- **LAN 1**: 2 PCs
- **LAN 2**: 2 Laptops
- **LAN 3**: 1 Server + 1 PC
- **LAN 4**: 2 Laptops

## 3. Subnetting Plan
The base Class B network `180.180.0.0/16` is subnetted into 8 equal-sized subnets using a `/19` prefix length.
- **Subnet Mask**: `255.255.224.0`
- **Subnet Allocations**:
  1. `180.180.0.0/19`   → LAN 1
  2. `180.180.32.0/19`  → Router0 – Router1 Link
  3. `180.180.64.0/19`  → LAN 2
  4. `180.180.96.0/19`  → Router1 – Router2 Link
  5. `180.180.128.0/19` → LAN 3
  6. `180.180.160.0/19` → Router2 – Router3 Link
  7. `180.180.192.0/19` → LAN 4
  8. `180.180.224.0/19` → Unused/Reserved

## 4. IP Addressing Table

| Device | Interface / Component | IP Address | Subnet Mask | Default Gateway |
| :--- | :--- | :--- | :--- | :--- |
| **Router0** | Fa0/1 (LAN 1) | 180.180.0.1 | 255.255.224.0 | N/A |
| | Fa0/0 (to R1) | 180.180.32.1 | 255.255.224.0 | N/A |
| **Router1** | Fa0/0 (to R0) | 180.180.32.2 | 255.255.224.0 | N/A |
| | Fa1/0 (to R2) | 180.180.96.1 | 255.255.224.0 | N/A |
| | Fa0/1 (LAN 2) | 180.180.64.1 | 255.255.224.0 | N/A |
| **Router2** | Fa0/0 (to R1) | 180.180.96.2 | 255.255.224.0 | N/A |
| | Fa1/0 (to R3) | 180.180.160.1 | 255.255.224.0 | N/A |
| | Fa0/1 (LAN 3) | 180.180.128.1 | 255.255.224.0 | N/A |
| **Router3** | Fa0/0 (to R2) | 180.180.160.2 | 255.255.224.0 | N/A |
| | Fa0/1 (LAN 4) | 180.180.192.1 | 255.255.224.0 | N/A |
| **LAN 1** | PC1, PC0 | DHCP Assigned | 255.255.224.0 | 180.180.0.1 |
| **LAN 2** | Laptop2, Laptop0 | DHCP Assigned | 255.255.224.0 | 180.180.64.1 |
| **LAN 3** | Server0, PC2 | DHCP Assigned | 255.255.224.0 | 180.180.128.1 |
| **LAN 4** | Laptop1, Laptop3 | DHCP Assigned | 255.255.224.0 | 180.180.192.1 |

*(Note: We will exclude the first 10 IPs in each subnet's DHCP pool for stable network device reservations like servers and gateways.)*

## 5. Router Configurations

### Router0
```ios
enable
configure terminal

! Interface Configuration
interface fa0/1
 ip address 180.180.0.1 255.255.224.0
 no shutdown
 exit

interface fa0/0
 ip address 180.180.32.1 255.255.224.0
 no shutdown
 exit

! Static Routing
ip route 180.180.64.0 255.255.224.0 180.180.32.2
ip route 180.180.96.0 255.255.224.0 180.180.32.2
ip route 180.180.128.0 255.255.224.0 180.180.32.2
ip route 180.180.160.0 255.255.224.0 180.180.32.2
ip route 180.180.192.0 255.255.224.0 180.180.32.2

! DHCP Configuration for LAN 1
ip dhcp excluded-address 180.180.0.1 180.180.0.10
ip dhcp pool LAN1_POOL
 network 180.180.0.0 255.255.224.0
 default-router 180.180.0.1
 dns-server 8.8.8.8
 exit
```

### Router1
```ios
enable
configure terminal

! Interface Configuration
interface fa0/0
 ip address 180.180.32.2 255.255.224.0
 no shutdown
 exit

interface fa1/0
 ip address 180.180.96.1 255.255.224.0
 no shutdown
 exit

interface fa0/1
 ip address 180.180.64.1 255.255.224.0
 no shutdown
 exit

! Static Routing
ip route 180.180.0.0 255.255.224.0 180.180.32.1
ip route 180.180.128.0 255.255.224.0 180.180.96.2
ip route 180.180.160.0 255.255.224.0 180.180.96.2
ip route 180.180.192.0 255.255.224.0 180.180.96.2

! DHCP Configuration for LAN 2
ip dhcp excluded-address 180.180.64.1 180.180.64.10
ip dhcp pool LAN2_POOL
 network 180.180.64.0 255.255.224.0
 default-router 180.180.64.1
 dns-server 8.8.8.8
 exit
```

### Router2
```ios
enable
configure terminal

! Interface Configuration
interface fa0/0
 ip address 180.180.96.2 255.255.224.0
 no shutdown
 exit

interface fa1/0
 ip address 180.180.160.1 255.255.224.0
 no shutdown
 exit

interface fa0/1
 ip address 180.180.128.1 255.255.224.0
 no shutdown
 exit

! Static Routing
ip route 180.180.0.0 255.255.224.0 180.180.96.1
ip route 180.180.32.0 255.255.224.0 180.180.96.1
ip route 180.180.64.0 255.255.224.0 180.180.96.1
ip route 180.180.192.0 255.255.224.0 180.180.160.2

! DHCP Configuration for LAN 3
ip dhcp excluded-address 180.180.128.1 180.180.128.10
ip dhcp pool LAN3_POOL
 network 180.180.128.0 255.255.224.0
 default-router 180.180.128.1
 dns-server 8.8.8.8
 exit
```

### Router3
```ios
enable
configure terminal

! Interface Configuration
interface fa0/0
 ip address 180.180.160.2 255.255.224.0
 no shutdown
 exit

interface fa0/1
 ip address 180.180.192.1 255.255.224.0
 no shutdown
 exit

! Static Routing
ip route 180.180.0.0 255.255.224.0 180.180.160.1
ip route 180.180.32.0 255.255.224.0 180.180.160.1
ip route 180.180.64.0 255.255.224.0 180.180.160.1
ip route 180.180.96.0 255.255.224.0 180.180.160.1
ip route 180.180.128.0 255.255.224.0 180.180.160.1

! DHCP Configuration for LAN 4
ip dhcp excluded-address 180.180.192.1 180.180.192.10
ip dhcp pool LAN4_POOL
 network 180.180.192.0 255.255.224.0
 default-router 180.180.192.1
 dns-server 8.8.8.8
 exit
```

## 6. End-Device DHCP Assignment Procedure
1. Open the **IP Configuration** or **Network Settings** of each PC/Laptop/Server.
2. Toggle the radio button from **Static** to **DHCP**. 
3. *Note*: The server device in LAN 3 can either be put on DHCP entirely, or kept as static using one of the statically excluded IP addresses (`180.180.128.2` is highly recommended since it falls into the `180.180.128.1 - 180.180.128.10` exclusion range).  

## 7. Verification Steps
1. **DHCP Validation**:
   - Run `show ip dhcp binding` on routers to review active IP leases securely distributed to network clients.
   - Run `ipconfig` (PC) to verify valid endpoint IPs under the `255.255.224.0` subnet structure. 
2. **Routing Validation**:
   - Run `show ip route` explicitly on each router to check routing table accuracy with `S` (Static) labeled remote subnets properly populated.
3. **End-to-End Testing**:
   - Ping from LAN 1 to LAN 2 (`ping 180.180.64.x`), LAN 3 (`ping 180.180.128.x`), and LAN 4 (`ping 180.180.192.x`)
   - Ping between all other LANs to manually verify full multi-hop reachability.
   - All diagnostic attempts should resolve optimally.

## 8. Conclusion
The network was successfully implemented using static routing architecture coupled with a structured Class B `/19` subnet scheme across 4 routers. Automatic IP assignments were actively introduced to the environment utilizing localized DHCP pools directly on Router interfaces—increasing device scalability, diminishing manual administrative burdens, and delivering complete, verified end-to-end multi-LAN visibility.
