# OSPF

The Open Shortest Path First (OSPF) protocol is a link-state protocol that builds and maintains the routing tables that are needed for IPv4 and IPv6 traffic.

The Shortest Path First (SPF) algorithm lives at the heart of OSPF. The algorithm, developed by Edsger Wybe Dijkstra in 1956, is used by OSPF to provide IP routing with high-speed convergence with a loop-free topology. OSPF provides its fast convergence using triggered, incremental updates by exchanging Link State Advertisements (LSAs) with neighboring OSPF routers. OSPF is a classless protocol, meaning it carries the subnet mask with all IP routes. It supports a structured hierarchical design model using autonomous systems, a backbone, and other areas.

Creates a neighbor relationship by exchanging hello packets

1. Propagates LSAs rather than routing table updates:
   
   1. Link: Router interface
   2. State: Description of an interface and its relationship to neighboring routers

3. Floods LSAs to all OSPF routers in the area, not just the directly connected routers
4. Pieces together all the LSAs that OSPF routers generate to create the OSPF link-state database
5. Uses the SPF algorithm to calculate the shortest path to each destination and places it in the routing table


OSPF uses a two-layer network hierarchy that has two primary elements:

1. Autonomous System (AS): An AS consists of a collection of networks under a common administration that shares a common routing strategy. An AS, which is sometimes called a domain, can be logically subdivided into multiple areas.
2. Area: An area is a grouping of contiguous networks. Areas are logical subdivisions of the AS.

Within each AS, a contiguous backbone area must be defined as Area 0. In the multiarea design, all other nonbackbone areas are connected to the backbone area.

## uses Link State Advertisements (LSAs)

Open Shortest Path First (OSPF) uses Link State Advertisements (LSAs) to share information about network topology and routing. There are several types of LSAs in OSPF:
1. Type 1 - Router LSAs: Generated by all routers, these LSAs describe the router's directly connected links (interfaces), their state (up or down), and the router's relationship to other OSPF routers on those links (DR, BDR, etc.).
2. Type 2 - Network LSAs: Generated by the Designated Router (DR) on multi-access networks (like Ethernet), these LSAs list all routers on the network segment that the DR can reach.
3. Type 3 - Summary LSAs: Generated by Area Border Routers (ABRs), these LSAs contain summary information about networks in other areas. They're used to reduce the amount of LSA information that needs to be sent between areas.
4. Type 4 - ASBR Summary LSAs: Generated by ABRs, these LSAs provide a route to an Autonomous System Boundary Router (ASBR) to routers in other areas.
5. Type 5 - Autonomous System External LSAs: Generated by ASBRs, these LSAs contain route information about destinations outside the OSPF autonomous system (like routes learned from BGP or static routes that are redistributed into OSPF).
6. Type 6 - Multicast OSPF LSAs: Used in Multicast OSPF (MOSPF) to carry multicast routing information. Note: This LSA type is not commonly used as MOSPF is rarely implemented.
7. Type 7 - NSSA External LSAs: Used in Not-So-Stubby Areas (NSSAs) to carry information about external routes. These LSAs are converted into Type 5 LSAs by the ABR before being sent to other areas.
8. Type 8 - External Attributes LSAs: Used in BGP-OSPF interaction to carry BGP attributes. These are not commonly used.
9. Type 9, 10, 11 - Opaque LSAs: Used to carry auxiliary information to OSPF routers. They're used in certain OSPF extensions, like traffic engineering.

## AREA

There are two types of routers from the configuration point of view:

1. Routers with single-area configuration: Internal routers, backbone routers, and Autonomous System Boundary Routers (ASBRs) that reside in one area.
2. Routers with a multiarea configuration: Area Border Routers (ABRs) and ASBRs that reside in more than one area.

Link-state routing protocols use a two-layer area hierarchy:

1. Backbone area: The primary function of this OSPF area is to quickly and efficiently move IP packets. Backbone areas interconnect with other OSPF area types. The OSPF hierarchical area structure requires that all areas connect directly to the backbone area. In the figure, directly connecting a device in Area 1 and Area 2 routers (R5 to R6, for example) is not allowed in OSPF. Interarea traffic must traverse the backbone. Generally, end users are not found within a backbone area, which is also known as OSPF Area 0.

2. Normal or nonbackbone area: The primary function of this OSPF area is to connect users and resources. Normal areas are usually set up according to functional or geographical groupings. By default, a normal area does not allow traffic from another area to use its links to reach other areas. All interarea traffic from other areas must cross a transit area such as Area 0. Normal areas can be of different types. Normal area types affect the amount of routing information that is propagated into the normal area. For example, instead of propagating all routes from the backbone area into a normal area, you could propagate only a default route.

OSPF router roles

1. Backbone routers
2. Internal routers
3. ABRs
4. ASBRs

## BASIC CONFIGURATION

![Creating a VLAN](https://raw.githubusercontent.com/deliawolf/OSPF/main/Untitled%20Diagram.drawio.png)

1.1.1.1
```
router ospf 1
 router-id 1.1.1.1
 area 0 authentication message-digest
 network 10.10.2.0 0.0.0.255 area 0
```
```
interface GigabitEthernet0/0
 ip address 10.10.2.2 255.255.255.0
 ip ospf message-digest-key 1 md5 yourpassword
```
2.2.2.2
```
router ospf 2
 router-id 2.2.2.2
 area 0 authentication message-digest
 network 10.10.1.0 0.0.0.255 area 0
 network 10.10.2.0 0.0.0.255 area 0
```
```
interface GigabitEthernet0/0
 ip address 10.10.2.1 255.255.255.0
 ip ospf message-digest-key 1 md5 yourpassword
!
interface GigabitEthernet0/1
 ip address 10.10.1.2 255.255.255.0
 ip ospf message-digest-key 1 md5 yourpassword
```
3.3.3.3 note*( unlike eigrp who need whole topology to be configured with auth, ospf only need it in some area only if needed so not required whole topology to be configured.)
```
router ospf 3
 router-id 3.3.3.3
 area 0 authentication message-digest
 network 10.10.1.0 0.0.0.255 area 0
 network 10.10.3.0 0.0.0.255 area 1
 network 10.10.4.0 0.0.0.255 area 2
```
```
interface GigabitEthernet0/0
 ip address 10.10.1.1 255.255.255.0
 ip ospf message-digest-key 1 md5 yourpassword
!
interface GigabitEthernet0/1
 ip address 10.10.4.1 255.255.255.0
!
interface GigabitEthernet0/2
 ip address 10.10.3.1 255.255.255.0
```
4.4.4.4
```
router ospf 4
 router-id 4.4.4.4
 network 10.10.3.0 0.0.0.255 area 1
```
5.5.5.5
```
router ospf 5
 router-id 5.5.5.5
 network 10.10.4.0 0.0.0.255 area 2
```
