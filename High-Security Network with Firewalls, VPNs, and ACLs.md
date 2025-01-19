# High-Security Network with Firewalls, VPNs, and ACLs  
Project Overview | 
This project simulates a secure enterprise network implementing multiple layers of security. The goal is to protect corporate resources while enabling secure remote access.

Cisco ASA Firewalls to secure network perimeters
Site-to-Site & Remote-Access VPNs for encrypted communication
ACLs (Access Control Lists) to restrict unauthorized access
AAA (Authentication, Authorization, Accounting) for secure user & device management

# Network Topology
Devices Used  
2 x Cisco ASA 5505 Firewalls (HQ Firewall, Branch Firewall)  
2 x Cisco 2911 Routers (HQ Router, Branch Router)  
1 x External Router (Simulating the Internet)  
1 x Windows Server (Acting as a RADIUS Server for AAA)  
End Devices (PCs, Laptops, Remote VPN Users)  
Logical Network Design  
HQ and Branch offices are connected via a Site-to-Site VPN.  
Remote workers connect securely using an SSL VPN.  
Firewalls enforce strict ACL policies to allow only authorized traffic.  
AAA Authentication ensures only legitimate users can access network resources.  
# Step 1: Configuring the ASA Firewall  
Basic ASA Configuration (HQ Firewall)  
hostname HQ-FW  
enable password cisco123  
interface gig0/0  
 nameif outside  
 security-level 0  
 ip address 203.0.113.1 255.255.255.252  
 no shutdown  
exit  

interface gig0/1  
 nameif inside  
 security-level 100  
 ip address 192.168.1.1 255.255.255.0  
 no shutdown  
exit  

Explanation:  
The outside interface connects to the internet (203.0.113.1).  
The inside interface is the internal corporate LAN (192.168.1.1).  
Security levels ensure outside traffic is untrusted, and inside traffic is fully trusted.  

NAT Configuration (HQ Firewall)  
object network INSIDE-NET  
 subnet 192.168.1.0 255.255.255.0  
nat (inside,outside) dynamic interface  

Explanation:  
Internal LAN traffic is translated to the public IP when accessing the internet.  
Firewall Rule: Allow Internal Traffic Outbound  
access-list OUTSIDE-IN extended deny ip any any  
access-group OUTSIDE-IN in interface outside  

Explanation:  
Blocks all incoming traffic by default.  
Additional rules will be added for VPN and permitted services.  

# Step 2: Configuring Site-to-Site VPN
A Site-to-Site VPN will securely connect HQ and Branch networks over the internet.  

- Step 1: Define ISAKMP Policy (HQ & Branch Firewalls)
 
crypto isakmp policy 10  
 encryption aes  
 hash sha256  
 authentication pre-share  
 group 5  
 lifetime 86400  
 
Explanation:  
Uses AES encryption and SHA-256 hashing for strong security.  

- Step 2: Configure Pre-Shared Keys  
crypto isakmp key VPNKEY address 198.51.100.2

Explanation:  
VPN key shared between HQ and Branch firewalls.

- Step 3: Define IPsec Tunnel

crypto ipsec transform-set VPN-SET esp-aes 256 esp-sha-hmac  
crypto map VPN-MAP 10 ipsec-isakmp  
 set peer 198.51.100.2  
 set transform-set VPN-SET  
 match address VPN-TRAFFIC  
 
Explanation:  
Defines IPsec encryption settings for the VPN.  

- Step 4: Apply VPN to Outside Interface

interface gig0/0  
crypto map VPN-MAP  
Explanation:  
Binds the VPN configuration to the outside interface.

- Step 5: Verify VPN Tunnel  

show crypto isakmp sa  
show crypto ipsec sa  

Expected Result: VPN tunnel should be active between HQ & Branch.  

# Step 3: Configuring SSL VPN for Remote Users  
Remote users will securely connect using SSL VPN (AnyConnect).  

1. Step 1: Enable WebVPN on the Firewall

webvpn
 enable outside
Explanation:

Enables SSL VPN on the outside interface.
Step 2: Configure User Authentication

username vpnuser password Cisco123
tunnel-group SSL-VPN general-attributes
 address-pool VPN-POOL
Explanation:

Defines remote user credentials.
Assigns an IP address from the VPN pool.
Step 3: Verify SSL VPN Connectivity
Remote users should be able to connect via AnyConnect VPN client.
Test access to internal corporate resources.
Step 4: Configuring ACLs for Network Security
Block Unwanted Traffic on HQ Firewall

access-list OUTSIDE-IN deny tcp any any eq 23
access-list OUTSIDE-IN permit ip any any
Explanation:

Blocks Telnet (port 23) while allowing all other traffic.
Restrict Internal Access on Branch Firewall

access-list INSIDE-OUT deny ip 192.168.2.0 0.0.0.255 any
access-list INSIDE-OUT permit ip any any
Explanation:

Prevents Branch users from accessing external networks.
Verify ACLs

show access-lists
Step 5: Configuring AAA Authentication
AAA ensures only authorized users can access network resources.

Enable AAA on the HQ Router

aaa new-model
aaa authentication login default group radius local
aaa authorization exec default group radius local
radius-server host 192.168.1.200 key SECRET123
Explanation:

Uses a RADIUS server for authentication.
If RADIUS is unavailable, local authentication is used.
Configure a RADIUS Server on Windows (NPS)
Install Network Policy Server (NPS).
Add the ASA Firewall as a RADIUS client.
Create a policy allowing VPN authentication.
Verify AAA Authentication

test aaa group radius admin password123
7️⃣ Conclusion & Next Steps
✅ Firewall rules implemented for network security
✅ Site-to-Site VPN established for secure branch connectivity
✅ SSL VPN enabled for remote workers
✅ ACLs restricting unauthorized traffic
✅ AAA authentication securing user access

Future Enhancements
📌 Deploy Intrusion Prevention System (IPS) for attack detection
📌 Implement SIEM logging & monitoring for security events
📌 Configure Failover Redundancy for high availability
