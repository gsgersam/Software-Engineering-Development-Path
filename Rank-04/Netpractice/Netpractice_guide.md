 <h1 align="center"> Netpactice</h1> 
 
## Basic notions

### Handle Ips and Masks

An IP address is composed of 32 bits, separated into four 8-bit octects. Consequently, each octet ranges from 0 to 255, with 0.0.0.0 being the minimum  and 255.255.255.255 the maximum. The latter cannot host any users; while theoretically possible, in practice 255.255.255.252 is the biggest mask that can allocate two usable hosts.

Similarly, IP address are divided into groups and ordered by ranges, meaning each range has a specific purpose.

| First Octet | Range | Class | Type/Use | Valid examples /Additional notes |
| :---: | :---: | :---: | :---: | :---: |
| 0 | 0.0.0.0 – 0.255.255.255 | Special | “This Network.”, Not assigned to hosts. Used for default routes or bootp/DHCP. | 0.0.0.0 Uses as “All IPs” or the "Any" address. |
| 1 | 1.0.0.0 – 126.255.255.255 | Class A | Public or Private addresses (10.0.0.0 Is private) | 10.0.01, 126.25.33.5 | 
| 127 | 127.0.0.0 – 127.255.255.255 | Special | Loopback. Used locally to internal testing | 127.0.0.1 is localhost | 
| 128 | 128.0.0.0 – 191.255.255.255 | Class B | Public or Private addresses (172.16.0.0 – 172.31.255.255 is private). | 172.16.5.1, 150.100.3.4 |
| 192 | 192.0.0.0 – 223.255.255.255 | Class C | Public or Private addresses (192.168.0.0/16 is private). | 224.0.0.1, 239.255.255.250 | 
| 224 | 224.0.0.0 – 239.255.255.255 | Class D | Multicast. Not assigned to individual hosts: Used for transmission to groups. | 224.0.0.1, 239.255.255.250 | 
| 240 | 240.0.0.0 – 255.255.255.255 | Class E | Reserved. Experimental use. Should not be used in public or private networks | 250.1.2.3 (invalid), 255.255.255.255 (broadcast) |

## Special ranges  

These IP addresses are reserved for private networks

- `10.0.0.0 – 10.255.255.255` 
- `172.16.0.0 – 172.31.255.255`
- `192.168.0.0 – 192.168.255.255`

The range form 127.0.0.0 to 127.255.255.255, is reserved for **loopback addresses**. Is used by computers to internal services, also knows as **"loopback interface"**. The most common of these is 127.0.0.1. These IP addresses do not travel through a physical network; the traffic send to a loopback address is managed completely inside of the operative system of that device.

 

## Masks 

The network-mask or subnet-mask. Decides which range of IP addresses belon to the same subnet. There are two ways to write a mask

- `Dot-decimal notation”: 255.255.255.0`

- `“Classless Inter-Domain Routing” or “CIDR”: /24`

| CIDR | Dot-decimal | Total IP addresses on subnet | Usable IP addresses | Subnets on class C |
| :---: | :---: | :---: | :---: | :---: |
| /32 | 255.255.255.255 | 1 | 0 | 256 (Idividual hosts) |
| /31 | 255.255.255.254 | 2 | 2* | 128 |
| /30 | 255.255.255.252 | 4 | 2 | 64 | 32 | 
| /28 | 255.255.255.240 | 16 | 14 | 16 | 
| /27 | 255.255.255.224 | 32 | 30 | 8 | 
| /26 | 255.255.255.192 | 64 | 62 | 4 | 
| /25 | 255.255.255.128 | 128 | 126 | 2 | 
| /24 | 255.255.255.0 | 256 | 254 | 1 |

 

<!-- El numero de IP-address por subnet is mas bajo que el total de numero de IP’s porque la primera direccion es reservada como network-address de la subnet y la ultima direccion es reservada como broadcast-address (trasmision de senales) -->
the number of usable IP address per subnet is lower than the total number of IPs because the first address is reserved as network-address, and the last one acts as broadcast-address.

i.e. for mask 255.255.255.252: 
network: 190.3.2.252 
broadcast: 190.3.2.255 
usable IP's: 190.3.2.253, 190.3.2.254 

 

### Switches:

- `Allows you to connect more than two devices to the same network.`

### Routers 

- `A router is an interface that allows the communication between different networks. A router has the ability to be part of several networks.`

![F.g 01](/documentations/study_guides/images/schema.png)
 
Here show it two different routes  

The destination is the network address where you want to send the packet, combined with the mask in CIDR notation (e.g 192.168.0.0./26). If you do not need a specific addres, you can use the default route 0.0.0.0./0.

The next hop is the IP address to the next router in the path that the packet must take to reach its destination.


## NetPractice: Technical Case Studies (1–10)
### Exercise 1 – Subnet Mismatch

#### Problem: 
- `Host A (192.168.1.2/24) cannot communicate with Host B (192.168.2.2/24). Similarly, C and D are on different networks.`

#### Analysis:
- `The hosts are separated by their Network IDs. A /24 mask (255.255.255.0) means the first three octets must match for Layer 2 communication. 192.168.1.x is a different network than 192.168.2.x.`

#### Solution:
- `Unify the subnets by matching the first three octets.`

    - `Set B to 192.168.1.3 /24.`

    - `Set D to 10.0.0.3 /24.`

### Exercise 2 – Invalid IP Ranges

#### Problem:
- `IPs 127.0.0.1 and 0.0.0.5 are causing connectivity failures.`

#### Analysis:
- `* 127.x.x.x is the Loopback Range (reserved for internal software testing).`

    - `0.x.x.x is a Reserved Range (non-routable).`

    - `Mismatched masks (/24 vs /16) create conflicting network boundaries.`

#### Solution:
- `Assign valid Private IPv4 addresses (RFC 1918) within the same subnet. Use 192.168.1.2 through 192.168.1.5 all with a /24 mask.`

### Exercise 3 – Inconsistent Subnet Masks- `* 127.x.x.x is the Loopback Range (reserved for internal software testing).`

-- `0.x.x.x is a Reserved Range (non-routable).`

-- `Mismatched masks (/24 vs /16) create conflicting network boundaries.`

#### Problem:
- `Host A is /24 and Host B is /25.`

#### Analysis:
- `A /25 mask (255.255.255.128) divides a /24 network into two smaller subnets. Even if the IPs seem close, the devices have different definitions of where the network ends and the broadcast address begins.`

#### Solution:
- `Standardize the masks. Set both devices to /24 to ensure they reside in the same broadcast domain.`

### Exercise 4 – One-Way Visibility

#### Problem:
- `A (10.0.0.1/16) can see B, but B (10.0.1.2/24) cannot see A.`

#### Analysis:
- `This is a Perception Error. Host A thinks the network is huge (10.0.0.0 to 10.0.255.255), so it sees B as local. Host B thinks the network is small (10.0.1.0 to 10.0.1.255), so it sees A as a "remote" address and tries to send traffic to a non-existent gateway.`

#### Solution:
- `Change Host B's mask to /16 so both devices agree on the network size.`

### Exercise 5 – Gateway Configuration

#### Problem:
- `Host (192.168.1.2/24) cannot reach the Router (192.168.2.1/24).`

#### Analysis:
- `A Host can only communicate with a Default Gateway that is inside its own subnet. Since the Host is on the .1.0 network and the Router is on the .2.0 network, they cannot "talk" to initiate routing.`

#### Solution:
- `Change the Host IP to 192.168.2.2/24 so it sits on the same segment as the Router.`

### Exercise 6 – Reserved Class E Addresses

#### Problem:
- `Using 240.x.x.x or 250.x.x.x prevents communication.`

#### Analysis:
- `These are Class E (Experimental) addresses. Standard TCP/IP stacks and routers are configured to drop this traffic by default as they are not meant for standard networking.`

#### Solution:
- `Use Private IP Ranges such as 10.0.0.0/8 or 192.168.0.0/16.`

### Exercise 7 – Subnet Waste (VLSM)

#### Problem:
- `Two devices are using a /24 mask.`

#### Analysis:
- `A /24 mask provides 254 usable IPs. Using this for a point-to-point connection (only 2 devices) is inefficient and wastes address space.`

#### Solution:
- `Implement VLSM (Variable Length Subnet Mask). Use a /30 mask (255.255.255.252), which provides exactly 2 usable IP addresses.`

### Exercise 8 – Public IP Overlap

#### Problem: 
- `Using Public IPs in a local LAN environment.`

#### Analysis:
- `If you use a Public IP (like 1.1.1.1) internally, your computer will never check the internet for the real 1.1.1.1. It will only look locally, breaking external connectivity for those specific addresses.`

#### Solution:
- `Use Private IPv4 ranges (10.x, 172.16.x, or 192.168.x). This allows for NAT (Network Address Translation) later.`

### Exercise 9 – Multi-Router Topology

#### Problem:
- `Complex network with multiple hosts (Meson, Ion, Gluon), two routers, and an Internet connection.`

#### Analysis:
- `This requires a structured IP Plan. You need a LAN for the hosts, a point-to-point link between routers, and a WAN link for the ISP.`

- `Solution: * LAN: 121.198.12.0/25 (Hosts and R1 Interface).`

    - `Link R1-R2: 41.243.18.252/30.`

    - `Routing: R2 needs a static route to the LAN via R1. Both routers need a Default Route (0.0.0.0/0) pointing towards the Internet.`

### Exercise 10 – Internet Uplink (WAN)

#### Problem:
- `Connectivity to the ISP is failing due to improper link addressing.`

- `Analysis: The interface between your Router and the ISP must share a small, dedicated subnet.`

- `Solution: Configure the WAN link with a /30 mask. Assign one IP to your Router (e.g., 200.0.0.1) and the other to the ISP Gateway (200.0.0.2).`

 ### Final Rules & Notes

- `The Rule of the Gateway: The Default Gateway address must belong to the same subnet as the host interface.`
- `The Rule of /30: Always use /30 for router-to-router links to conserve IP space.`
- `The Rule of Private IPs: Never use public IP ranges for internal LANs to avoid routing conflicts.`

#### Correction Ledger:
- `In Exercise 4, remember that communication is bidirectional. If A can talk to B but B can't talk to A, the connection is functionally dead.`
