# PCampus Campus Network
Designed and implemented in Cisco Packet Tracer 


## Network Topology

![PCampus Network Topology](./topology.png)


## Project Overview

The PCampus campus network is designed and implemented in Cisco Packet Tracer. It serves multiple zones including Boys Hostel (Blocks A/B/C), Girls Hostel, Masters Hostel, Department, CIT/Library, and a DMZ server zone. The network provides internet access via dual ISP, internal web/DNS hosting, VLAN segmentation, dynamic routing via OSPF, NAT/PAT, and perimeter security via ACLs.



## Device Summary

| Device | Model | Role |
|--------|-------|------|
| sachin1(BorderRouter) | Cisco 2911 | Border Router — Dual ISP, OSPF |
| GE Switch | Cisco 2960-24TT | L2 Switch between sachin1 and sachin2 |
| sachin2(Firewall Router) | Cisco 2911 | Firewall — NAT, ACL, AAA |
| CentralSwitch | Cisco 3560-24PS | Core L3 Switch — VLANs, OSPF, DHCP |
| BoysHostelSwitch | Cisco 3560-24PS | L3 Aggregation — Boys Hostel |
| DeptSwitch | Cisco 3560-24PS | L3 Switch — Department & RFID |
| CITSwitch | Cisco 2960-24TT | L2 Switch — CIT & Library |
| MastersHostelSwitch | Cisco 2960-24TT | L2 Switch — Masters Hostel |
| GirlsHostelSwitch | Cisco 2960-24TT | L2 Switch — Girls Hostel |
| ServerSwitch | Cisco 2960-24TT | L2 Switch — DMZ Servers |
| BlockA/B/CSwitch | Cisco 2960-24TT | L2 Access — Boys Blocks |
| Floor1/2/3Switch | Cisco 2960-24TT | L2 Access — Masters Floors |




## 3. IP Addressing Plan

### 3.1 WAN & Core Links

| Interface | IP Address | Connected To |
|---|---|---|
| sachin1 Gig0/0 | 203.0.113.2/30 | NTC Internet Server (40 Mbps) |
| sachin1 Gig0/1 | 198.51.100.2/30 | Vianet Internet Server (80 Mbps) |
| sachin1 Gig0/2 | 10.10.10.1/30 | sachin2 Gig0/0 |
| sachin1 Loopback0 | 1.1.1.1/24 | Simulated internet reachability |
| sachin2 Gig0/0 | 10.10.10.2/30 | sachin1 Gig0/2 (NAT Outside) |
| sachin2 Gig0/1 | 10.10.20.1/30 | CentralSwitch Fa0/1 (NAT Inside) |
| CentralSwitch Fa0/1 | 10.10.20.2/30 | sachin2 Gig0/1 |
| CentralSwitch Fa0/7 | 192.168.18.1/24 | ServerSwitch — DMZ Routed Port |

### 3.2 VLAN SVIs on CentralSwitch

| VLAN | Name | Subnet | Gateway |
|---|---|---|---|
| 4 | RFID | 10.100.4.0/23 | 10.100.4.1 |
| 20 | CIT_Library | 10.100.20.0/24 | 10.100.20.1 |
| 51 | Boys_Block_A | 10.100.51.0/24 | 10.100.51.1 |
| 52 | Boys_Block_B | 10.100.52.0/24 | 10.100.52.1 |
| 53 | Boys_Block_C | 10.100.53.0/24 | 10.100.53.1 |
| 56 | Boys_AP_A | 10.100.56.0/24 | 10.100.56.1 |
| 57 | Boys_AP_B | 10.100.57.0/24 | 10.100.57.1 |
| 58 | Boys_AP_C | 10.100.58.0/24 | 10.100.58.1 |
| 60 | Girls_Hostel | 10.100.60.0/23 | 10.100.60.1 |
| 70 | Masters_Hostel | 10.100.70.0/24 | 10.100.70.1 |
| 80 | Department | 10.100.80.0/23 | 10.100.80.1 |

### 3.3 DMZ Servers — Static Public IPs

| Server | IP Address | Gateway | DNS |
|---|---|---|---|
| Web Server | 192.168.18.2 | 192.168.18.1 | 192.168.18.3 |
| Primary DNS | 192.168.18.3 | 192.168.18.1 | 192.168.18.3 |
| Secondary DNS | 192.168.18.4 | 192.168.18.1 | 192.168.18.3 |

### 3.4 Simulated Internet Servers

| Server | IP Address | Gateway | Connected To |
|---|---|---|---|
| NTC Internet Server | 203.0.113.1/30 | 203.0.113.2 | sachin1 Gig0/0 |
| Vianet Internet Server | 198.51.100.1/30 | 198.51.100.2 | sachin1 Gig0/1 |


## 4. Device Configurations

### 4.1 sachin1 — Border Router
**Credentials:** Enable secret: `class` | Console: `cisco` | Telnet: `network`

**Interfaces:**
- Gig0/0: `203.0.113.2/30` — NTC ISP
- Gig0/1: `198.51.100.2/30` — Vianet ISP
- Gig0/2: `10.10.10.1/30` — to sachin2
- Loopback0: `1.1.1.1/24` — simulated internet

**Routing table highlights:**
- Primary default route via NTC: `S* 0.0.0.0/0 [1/0] via 203.0.113.1`
- Backup default route via Vianet: `S* 0.0.0.0/0 [10/0] via 198.51.100.1`
- All internal subnets learned via OSPF from sachin2






### 4.2 sachin2 — Firewall Router
**Credentials:** Enable secret: `class` | Console: `cisco` | Telnet: `network`

**Interfaces:**
- Gig0/0: `10.10.10.2/30` — `ip nat outside`
- Gig0/1: `10.10.20.1/30` — `ip nat inside`

**NAT Configuration:**
```
ip access-list standard NAT_ACL
permit 10.0.0.0 0.255.255.255
permit 192.168.18.0 0.0.0.255
ip nat inside source list NAT_ACL interface GigabitEthernet0/0 overload
```
**ACL — OUTSIDE_IN (applied inbound on Gig0/0):**
```
ip access-list extended OUTSIDE_IN
permit ospf host 10.10.10.1 host 224.0.0.5
permit ospf host 10.10.10.1 host 10.10.10.2
permit tcp any host 192.168.18.2 eq 80
permit tcp any host 192.168.18.2 eq 443
permit udp any host 192.168.18.3 eq 53
permit tcp any host 192.168.18.3 eq 53
permit udp any host 192.168.18.4 eq 53
permit icmp any host 192.168.18.2 echo
permit icmp any host 192.168.18.3 echo
permit icmp any host 192.168.18.4 echo
permit icmp any any echo-reply
permit icmp any any
deny ip any any
```

**Routing table highlights:**
- Default route learned via OSPF: `O*E2 0.0.0.0/0 [110/1] via 10.10.10.1`
- All internal VLANs learned via OSPF from CentralSwitch
- All subnets via 10.10.20.2 on Gig0/1

---


### 4.3 CentralSwitch — Core L3 Switch
**Credentials:** Enable secret: `class` | Console: `cisco` | Telnet: `network`

**Key configuration:**
```
ip routing
interface FastEthernet0/1
no switchport
ip address 10.10.20.2 255.255.255.252
interface FastEthernet0/7
no switchport
ip address 192.168.18.1 255.255.255.0
router ospf 1
network 10.10.20.0 0.0.0.3 area 0
network 192.168.18.0 0.0.0.255 area 0
network 10.100.4.0 0.0.1.255 area 0
! ... all VLAN subnets advertised
```

**Port Mapping:**

| Port | Connected To | Mode |
|---|---|---|
| Fa0/1 | sachin2 Gig0/1 | Routed Port |
| Fa0/2 | BoysHostelSwitch | Trunk (dot1q) |
| Fa0/3 | DeptSwitch | Trunk (dot1q) |
| Fa0/4 | CITSwitch | Trunk (dot1q) |
| Fa0/5 | MastersHostelSwitch | Trunk (dot1q) |
| Fa0/6 | GirlsHostelSwitch | Trunk (dot1q) |
| Fa0/7 | ServerSwitch (DMZ) | Routed Port |

**DHCP Pools:** One pool per VLAN. Each assigns correct subnet IP, VLAN gateway, and DNS server `192.168.18.3`.

---

### 4.4 BoysHostelSwitch

**VLANs:** 51, 52, 53, 56, 57, 58 | `ip routing` enabled

| Port | Connected To | Mode |
|---|---|---|
| Fa0/1 | CentralSwitch Fa0/2 | Trunk (uplink) |
| Fa0/2 | BlockASwitch | Trunk |
| Fa0/3 | Block A AP | Access VLAN 56 |
| Fa0/4 | BlockBSwitch | Trunk |
| Fa0/5 | Block B AP | Access VLAN 57 |
| Fa0/6 | BlockCSwitch | Trunk |
| Fa0/7 | Block C AP | Access VLAN 58 |

---

### 4.5 Block Switches

| Switch | VLAN | Uplink | Access Ports |
|---|---|---|---|
| BlockASwitch | 51 | Fa0/3 trunk → BoysHostelSwitch Fa0/2 | Fa0/1: PC-A1 |
| BlockBSwitch | 52 | Fa0/3 trunk → BoysHostelSwitch Fa0/4 | Fa0/1: PC-B1, Fa0/4: FSU AP |
| BlockCSwitch | 53 | Fa0/3 trunk → BoysHostelSwitch Fa0/6 | Fa0/1: PC-C1, Fa0/4: Canteen AP |

---

### 4.6 DeptSwitch

**VLANs:** 4 (RFID), 80 (Department) | `ip routing` enabled

| Port | Connected To | Mode |
|---|---|---|
| Fa0/1 | CentralSwitch Fa0/3 | Trunk (uplink) |
| Fa0/2 | Dept PC1 | Access VLAN 80 |
| Fa0/3 | Dept PC2 | Access VLAN 80 |
| Fa0/4 | RFID Reader | Access VLAN 4 |
| Fa0/5 | Dept AP | Access VLAN 80 |

---

### 4.7 CITSwitch

**VLAN:** 20

| Port | Connected To | Mode |
|---|---|---|
| Fa0/1 | CentralSwitch Fa0/4 | Trunk (uplink) |
| Fa0/2 | CIT AP | Access VLAN 20 |
| Fa0/3 | Library AP | Access VLAN 20 |

---

### 4.8 MastersHostelSwitch + Floor Switches

**VLAN:** 70

| Port | Connected To | Mode |
|---|---|---|
| Fa0/4 | CentralSwitch Fa0/5 | Trunk (uplink) |
| Fa0/1 | Floor1Switch | Trunk |
| Fa0/2 | Floor2Switch | Trunk |
| Fa0/3 | Floor3Switch | Trunk |

Each Floor Switch: `Fa0/1` trunk uplink, `Fa0/2` access VLAN 70 → PC8/PC9/PC10

---

### 4.9 GirlsHostelSwitch

**VLAN:** 60 | Trunk uplink to CentralSwitch Fa0/6 | Access ports for APs in VLAN 60

---

### 4.10 ServerSwitch — DMZ

| Port | Connected To |
|---|---|
| Fa0/1 | CentralSwitch Fa0/7 (DMZ routed port) |
| Fa0/4 | Web Server (192.168.18.2) |
| Fa0/5 | Primary DNS (192.168.18.3) |
| Fa0/6 | Secondary DNS (192.168.18.4) |



## 5. Services

### 5.1 DNS — Primary Server (192.168.18.3)

- DNS Service: **ON**
- A Record: `pcampus.local` → `192.168.18.2`
- A Record: `www.pcampus.local` → `192.168.18.2`

### 5.2 DNS — Secondary Server (192.168.18.4)

- DNS Service: **ON**
- A Record: `pcampus.local` → `192.168.18.2`
- A Record: `www.pcampus.local` → `192.168.18.2`

### 5.3 Web Server (192.168.18.2)
- HTTP Service: **ON**
- HTTPS Service: **ON**
- `index.html`: Custom PCampus welcome page
- Accessible internally: `http://www.pcampus.local`
- Accessible from internet: `http://192.168.18.2` (static public IP)

>  DMZ servers use static public IPs directly. No NAT/port forwarding is needed for the DMZ zone.

### 5.4 DHCP (on CentralSwitch)

One pool per VLAN. Each pool assigns:
- IP from the correct VLAN subnet
- Default gateway: the VLAN SVI IP
- DNS server: `192.168.18.3`



## 6. Routing — OSPF

OSPF process 1, area 0 runs across sachin1, sachin2, and CentralSwitch.

| Router | Role | Networks Advertised |
|---|---|---|
| sachin1 | ASBR | 10.10.10.0/30, default route (originate) |
| sachin2 | ABR | 10.10.10.0/30, 10.10.20.0/30 |
| CentralSwitch | Internal | 10.10.20.0/30, 192.168.18.0/24, all VLAN subnets |


## 7. Security

### 7.1 ACL — OUTSIDE_IN on sachin2

Applied **inbound on Gig0/0** (outside/internet-facing interface).

| Seq | Protocol | Source | Destination | Port | Action | Purpose |
|---|---|---|---|---|---|---|
| 10 | OSPF | 10.10.10.1 | 224.0.0.5 | — | PERMIT | OSPF multicast |
| 20 | OSPF | 10.10.10.1 | 10.10.10.2 | — | PERMIT | OSPF unicast |
| 30 | TCP | any | 192.168.18.2 | 80 | PERMIT | HTTP to web server |
| 40 | TCP | any | 192.168.18.2 | 443 | PERMIT | HTTPS to web server |
| 50 | UDP | any | 192.168.18.3 | 53 | PERMIT | DNS to primary |
| 60 | TCP | any | 192.168.18.3 | 53 | PERMIT | DNS TCP to primary |
| 70 | UDP | any | 192.168.18.4 | 53 | PERMIT | DNS to secondary |
| 80 | ICMP | any | 192.168.18.2/3/4 | echo | PERMIT | Ping to DMZ only |
| 90 | ICMP | any | any | echo-reply | PERMIT | Ping replies inbound |
| 100 | ICMP | any | any | any | PERMIT | Traceroute hops |
| 110 | IP | any | any | any | DENY | Block everything else |

### 7.2 NAT/PAT on sachin2

- NAT_ACL permits all `10.0.0.0/8` and `192.168.18.0/24` traffic
- PAT overload on Gig0/0 — all internal hosts share the WAN IP `203.0.113.2`
- DMZ servers (`192.168.18.x`) have static public IPs — **no NAT required for DMZ**

### 7.3 Device Passwords

| Credential | Value |
|---|---|
| Enable Secret | `class` |
| Console Password | `cisco` |
| VTY (Telnet) Password | `network` |

## 8. Verification & Test Results

### 8.1 Internal Campus PC Tests

| Test | Command | Expected | Result |
|---|---|---|---|
| DHCP assignment | `ipconfig` | Correct VLAN IP + gateway + DNS | PASS |
| Gateway ping | `ping <gateway>` | Reply from gateway | PASS |
| Inter-VLAN routing | `ping <other VLAN IP>` | Reply | PASS |
| Web server ping | `ping 192.168.18.2` | Reply | PASS |
| DNS resolution | `nslookup pcampus.local` | Returns 192.168.18.2 | PASS |
| Web browsing (IP) | `http://192.168.18.2` | Campus page loads | PASS |
| Web browsing (domain) | `http://www.pcampus.local` | Campus page loads | PASS |
| Internet ping | `ping 203.0.113.1` | Reply from NTC server | PASS |
| Traceroute internet | `tracert 203.0.113.1` | 4 hops, trace complete | PASS |
| Traceroute unknown IP | `tracert 2.2.2.2` | 3 hops then timeout (expected) | PASS |

### 8.2 From NTC / Vianet Internet Servers


| Test | Command | Expected | Result |
|---|---|---|---|
| Ping web server | `ping 192.168.18.2` | Reply | PASS |
| Ping DNS server | `ping 192.168.18.3` | Reply | PASS |
| Ping private VLAN | `ping 10.100.20.1` | Blocked by ACL | PASS |
| DNS lookup | `nslookup pcampus.local` | Resolves to 192.168.18.2 | PASS |
| Web access | `http://192.168.18.2` | Campus page loads | PASS |




## Useful verification commands 
```
show ip route
show ip ospf neighbor
show ip nat translations
show ip nat statistics
show ip access-lists
show vlan brief
show interfaces trunk
show ip interface brief
show ip dhcp binding
show running-config
```


## 10. Conclusion

This project successfully demonstrates the design and implementation of a multi-zone campus network for PCampus using Cisco Packet Tracer. The network achieves full connectivity across all zones through VLAN segmentation, inter-VLAN routing via OSPF, and centralized L3 switching. Internet access is provided through dual ISP links with automatic metric-based failover, and all internal hosts receive IP configuration dynamically via DHCP.

The DMZ hosts internal web and DNS services with static public IPs, accessible campus-wide via the domain `pcampus.local` and directly reachable from the simulated internet as `192.168.18.2`. Security is enforced at the perimeter through NAT/PAT on sachin2, which masks all internal private addresses, and an extended ACL (OUTSIDE_IN) that permits only HTTP, HTTPS, DNS, and ICMP to DMZ servers while blocking all unsolicited access to internal private VLANs.

All verification tests passed, confirming end-to-end connectivity, correct OSPF routing, functional NAT, DNS resolution, web hosting, and effective ACL enforcement.