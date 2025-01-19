# ISP-Level Network with BGP & MPLS
Project Overview
This project simulates an Internet Service Provider (ISP) network that connects multiple businesses and customers. The network will demonstrate:
BGP (Border Gateway Protocol) for inter-domain routing & internet connectivity  
MPLS (Multiprotocol Label Switching) for efficient traffic engineering & QoS optimization  
Load Balancing for redundancy and fault tolerance across multiple connections  

# Network Topology
Devices Used  
3 x Cisco 7200 Routers (ISP Core: R1, R2, R3)  
2 x Cisco 2911 Routers (Customer Edge Routers: CE1, CE2)  
1 x External BGP Router (Simulating the Internet)  
End devices (PCs and servers)  
Logical Network Design  
ISP Core (R1, R2, R3) uses MPLS for internal backbone connectivity.  
Customer Edge Routers (CE1, CE2) connect customers via BGP.  
Load balancing & redundancy implemented across multiple ISP links.  

# Step 1: Configuring BGP for Internet Connectivity  
Each ISP router will participate in BGP (Autonomous System 100) to connect customers to the internet.

BGP Configuration on ISP Router (R1)  
router bgp 100  
 bgp log-neighbor-changes  
 neighbor 192.168.1.2 remote-as 200  
 neighbor 192.168.1.2 description Internet  
 network 203.0.113.0 mask 255.255.255.0  
 
Explanation:  
R1 connects to the internet (AS 200) via 192.168.1.2.
Advertises the 203.0.113.0/24 network to the internet.  

BGP Configuration on Customer Edge Router (CE1 - AS 300)  
router bgp 300  
 bgp log-neighbor-changes  
 neighbor 10.0.0.1 remote-as 100  
 network 192.168.10.0 mask 255.255.255.0  
 
 Explanation:  
CE1 (AS 300) connects to ISP (AS 100) via 10.0.0.1.  
Advertises the 192.168.10.0/24 network to ISP.  

Verify BGP Neighbors  
show ip bgp summary  

# Step 2: Configuring MPLS for Traffic Engineering  
Enable MPLS on ISP Core Routers (R1, R2, R3)  
mpls ip  
ip cef  
router ospf 1  
 network 10.0.0.0 0.0.0.255 area 0  
 
Explanation:  
Enables MPLS forwarding across ISP core routers.
Uses OSPF as the Interior Gateway Protocol (IGP) for dynamic routing.  

Configure MPLS LDP (Label Distribution Protocol) on ISP Core  
mpls label protocol ldp  
mpls ldp router-id loopback0 force  

Explanation:  
Uses LDP (Label Distribution Protocol) for MPLS label assignment.
Assigns router ID to Loopback 0 for stability.  

Verify MPLS Configuration  
show mpls ldp neighbors  
show mpls forwarding-table  

# Step 3: Configuring Load Balancing & Redundancy  
Implementing ECMP (Equal-Cost Multi-Path Routing) on ISP Core  
router ospf 1  
 maximum-paths 4  
 
Explanation:  
Allows OSPF to load balance traffic across 4 equal-cost paths.  
Configuring BGP Load Balancing with Multiple Paths
router bgp 100  
 bgp bestpath as-path multipath-relax  
 
Explanation:  
Enables BGP multipath routing to balance traffic across multiple ISP links.  

Verify Load Balancing  
show ip route  
show ip bgp paths  
# Step 4: Testing & Validation  
Ping Between Customer Networks (CE1 to CE2 via MPLS)   
ping 192.168.20.1 source 192.168.10.1  
Expected Result: Successful ping via MPLS backbone.  

Trace Route to Internet from Customer (CE1 to Internet)  
traceroute 8.8.8.8  
Expected Result: Traffic should pass through ISP Core (R1, R2, R3) and reach the internet.  

Conclusion & Next Steps  
✅ BGP successfully configured for customer internet access  
✅ MPLS backbone established for traffic engineering  
✅ Load balancing & redundancy implemented for fault tolerance  
