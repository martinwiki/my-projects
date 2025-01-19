# Project Scope

This project simulates an enterprise with two branch offices and a headquarters (HQ). The network will implement:  
✅ OSPF for dynamic routing  
✅ VLAN segmentation for better security and organization  
✅ Inter-VLAN routing for communication between VLANs  
✅ Redundancy with HSRP and EtherChannel  

# Network Topology

HQ: Acts as the main site, hosting core services (DNS, DHCP, Web, etc.).  
Branch 1: Smaller office, connected via an ISP router to HQ.  
Branch 2: Similar to Branch 1 but uses a different ISP link.  
ISP: Simulated backbone connecting all sites.  

# Step 1: Setting Up the Basic Network  
Devices to Use:  
3 x Cisco 2911 Routers (HQ, Branch 1, Branch 2)  
3 x Cisco 3560 Switches (One per site)  
End devices (PCs, Servers, Phones)  
ISP Router to simulate internet connectivity  
IP Addressing Plan (Example Subnetting):  

HQ: 192.168.1.0/24  
Branch 1: 192.168.2.0/24  
Branch 2: 192.168.3.0/24  
WAN Links: /30 subnets for router-to-router connections  

# Step 2: Configuring VLANs on HQ Switch  
Create VLANs for different departments:  

enable  
configure terminal  
vlan 10  
name HR  
vlan 20  
name IT  
vlan 30  
name Sales  
vlan 40  
name Finance  
exit  

Assign VLANs to switch ports:  

interface range fa0/1 - 5  
switchport mode access  
switchport access vlan 10  
exit  

Configure trunk links between the HQ switch and router:  

interface fa0/24  
switchport mode trunk  
exit  

# Step 3: Configuring Inter-VLAN Routing on HQ Router  

Create subinterfaces for each VLAN:  

interface gig0/0.10  
encapsulation dot1Q 10  
ip address 192.168.1.1 255.255.255.0  
exit  

interface gig0/0.20  
encapsulation dot1Q 20  
ip address 192.168.1.2 255.255.255.0  
exit  

interface gig0/0.30  
encapsulation dot1Q 30  
ip address 192.168.1.3 255.255.255.0  
exit  

Enable IP routing:  

ip routing  

# Step 4: Configuring OSPF for Dynamic Routing  

Enable OSPF and assign networks:  

router ospf 1  
network 192.168.1.0 0.0.0.255 area 0  
network 192.168.2.0 0.0.0.255 area 0  
network 192.168.3.0 0.0.0.255 area 0  
exit  

Repeat similar OSPF configurations on Branch 1 and Branch 2 routers.  

# Step 5: Configuring Redundancy with HSRP  

On HQ Router (Primary):  

interface gig0/0.10  
standby 1 ip 192.168.1.254  
standby 1 priority 110  
standby 1 preempt  
exit  

On Backup Router:  

interface gig0/0.10  
standby 1 ip 192.168.1.254  
standby 1 priority 90  
standby 1 preempt  
exit  

# Step 6: Configuring EtherChannel for High Availability  

Create an EtherChannel between the HQ switch and router:  

interface range fa0/23 - 24  
channel-group 1 mode active  
exit  

interface port-channel 1  
switchport mode trunk  
exit  
