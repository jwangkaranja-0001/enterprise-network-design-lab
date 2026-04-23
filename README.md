## Introduction

This project demonstrates the design and implementation of a small-scale enterprise campus network spanning three floors. It supports three departmental VLANs (HR, Sales, IT), centralized routing, and a secure Layer 2 switching infrastructure.

The network was designed to provide segmentation, scalability, security, and manageability through VLANs, trunk links, Spanning Tree Protocol (STP), router-on-a-stick inter-VLAN routing, DHCP services, and switch management via a dedicated management VLAN.

## Network Design Overview (Topology)

### Infrastructure Design

The network consists of the following components:

| Device | Role |
| --- | --- |
| 12 PCs | End-user devices (4 per switch) |
| SW-ACCESS-1 | Floor 1 Access Switch |
| SW-ACCESS-2 | Floor 2 Access Switch |
| SW-ACCESS-3 | Floor 3 Access Switch |
| SW-DIST-1 | Distribution Switch (Layer 2 aggregation + STP root) |
| R-CORE-1 | Router (Inter-VLAN routing using Router-on-a-Stick) |

![image.png](attachment:56ac37fe-bfa1-41f0-ac58-69e3da9a65fc:image.png)

### 2.2 Design Principles

- Hierarchical network design (Access → Distribution → Core)
- VLAN segmentation per department
- Centralized routing at the core router
- Redundant trunk links for resilience

## 3. VLAN & IP Addressing Design

### 3.1 VLAN Structure

| VLAN ID | Name | Purpose |
| --- | --- | --- |
| 10 | HR | Human Resources Department |
| 20 | SALES | Sales Department |
| 30 | IT | IT Department |
| 99 | MGMT | Network Device Management |

### 3.2 IP Addressing Scheme

| VLAN | Subnet | Default Gateway |
| --- | --- | --- |
| 10 | 192.168.10.0/24 | 192.168.10.1 |
| 20 | 192.168.20.0/24 | 192.168.20.1 |
| 30 | 192.168.30.0/24 | 192.168.30.1 |
| 99 | 192.168.99.0/24 | 192.168.99.1 |

## 4. VLAN Implementation

VLANs were created consistently across all switches to ensure uniform Layer 2 segmentation.

### Configuration Applied

Enter global configuration mode

`enable`

`configure terminal`

Create logical separation with vlans across all the switches

Start with naming the switch on the terminal

`hostname SW-ACCESS-1`

`no ip domain-lookup`

(This names the switch and then stops errors from hanging on the CLI)

Named all switches and the router. 

`vlan 10`

`name HR`

`vlan 20`

`name SALES`

`vlan 30`

`name IT`

`vlan 99`

`name MGMT`

### Verification

- `show vlan brief` confirmed VLAN consistency across all switches

![Switch Distribution 1](attachment:fad47fb9-202c-4e56-bd58-dbeb56b390f7:image.png)

Switch Distribution 1

![Switch Access 2](attachment:c5cc1681-5eac-435e-a4e8-9ea1362dde78:image.png)

Switch Access 2

![Switch Access 3](attachment:053d4684-d890-4a33-85fe-247a17047fd9:image.png)

Switch Access 3

## 5. Access Port Assignment

Each switch was configured to assign end devices to their respective VLANs based on departmental grouping.

### Design Outcome

- HR devices isolated in VLAN 10
- Sales devices isolated in VLAN 20
- IT devices isolated in VLAN 30

Example configuration

`interface fastEthernet0/1`

`switchport mode access`

`switchport access vlan 10`

`description MW3-HR-01`

![Switch Access 1](attachment:e55e08c0-fa90-4da5-b4bc-7e4699e255d9:image.png)

Switch Access 1

![Switch Access 2](attachment:e3d75a80-dabe-4433-932b-4955bb54a37f:image.png)

Switch Access 2

![Switch access 3](attachment:e76467f9-3f0d-4a34-8368-1fed8809ba67:image.png)

Switch access 3

## 6. Trunking Configuration

Trunk links were configured between access and distribution switches to carry multiple VLANs.

### Design Justification

- Enables VLAN propagation across switches
- Supports inter-floor communication via routing layer
- Redundancy introduced through multiple trunk links

Configured trunks between the different switches 

`interface fastEthernet0/5`

`switchport mode trunk`

`switchport trunk allowed vlan 10,20,30,99`

`description TRUNK TO SW-ACCESS-2`

### Verification

`show interfaces trunk`

![Switch Access Two - Configure trunks](attachment:1b5d0bcb-a065-4a63-8818-c74b2a818218:image.png)

Switch Access Two - Configure trunks

![Switch Access 1 - Verify Trunks](attachment:138bd793-e937-4e13-b874-e2c01a01385a:image.png)

Switch Access 1 - Verify Trunks

![Switch Access 3 - Configured Trunks and verified results](attachment:a5f0a42c-33ec-4037-b4d6-2b8a790bf64d:image.png)

Switch Access 3 - Configured Trunks and verified results

![Switch Distribution 1 - Configured trunks](attachment:475f975f-eec7-43e3-b609-40363ee50ed0:image.png)

Switch Distribution 1 - Configured trunks

We have multiple trunks between access switches, and this creates redundancy, which means STP, Spanning tree will block one path. 

STP is doing

1. Detecting loops 
2. Blocking one link
3. Keeping backup path ready

### STP Behavior

Spanning Tree Protocol automatically prevents loops by blocking redundant paths while maintaining backup links.

## 7. Spanning Tree Protocol (STP)

### Configuration

SW-DIST-1 was configured as the root bridge:

`spanning-tree vlan 10,20,30,99 root primary`

### Role of STP

- Prevents Layer 2 switching loops
- Ensures single active path per VLAN
- Maintains backup redundant links

### Result

SW-DIST-1 acts as the central switching decision point for the campus network.

![Verification results of: show spanning-tree before configuration](attachment:972a420a-7a9c-4897-b105-78f025e4e762:image.png)

Verification results of: show spanning-tree before configuration

![Results of verification of: show spanning-tree after configuration](attachment:0807ab4d-0694-4a54-928c-98cfc37f8a40:image.png)

Results of verification of: show spanning-tree after configuration

We get to see it was able to configure SW-DIST-1 into becoming the root, and in the verification results, it also shows that this bridge is root.

#### For full control

Enter global configuration mode then: 

`spanning-tree vlan 1 root primary`

STP is:

- Keeping one path active
- Blocking another
- Preventing loops

SW-DIST-1 is now:

- Central decision maker
- Closest to router
- Best traffic path

## 8. Inter-VLAN Routing (Router-on-a-Stick)

Router-on-a-stick was implemented using subinterfaces on R-CORE-1 to enable communication between VLANs.

### Concept

- Each VLAN is assigned a logical subinterface
- 802.1Q tagging separates traffic logically
- Single physical interface carries all VLAN traffic

### Outcome

- Successful inter-VLAN communication verified via ping tests

![Trunking on SW-DIST-1](attachment:7a5de08b-ae6a-4d88-b5df-e0b9d01b2dc4:image.png)

Trunking on SW-DIST-1

![Trunking on R-CORE-1](attachment:1dcac908-df3d-4026-8813-7b039e1435d3:image.png)

Trunking on R-CORE-1

![Network after no shutdown on R-CORE-1](attachment:76d569a7-9a33-4aa5-88d0-07c3088d7bbe:image.png)

Network after no shutdown on R-CORE-1

![801.1Q Tagging on vlans to separate traffic](attachment:38465f16-85cc-4cd2-be8e-8defaa0f9216:image.png)

801.1Q Tagging on vlans to separate traffic

![Tagging to separate traffic logically](attachment:ac9d5c76-eb8c-41ca-ac2d-11f9a2f423d6:image.png)

Tagging to separate traffic logically

### Testing Connectivity after manual configuration

Manually set ip addresses on 3 pcs and then ran ping across different vlans. 

Results showed ping results successful. 

![image.png](attachment:b354d9b2-5348-4cef-a7d1-f96bb4cc865a:image.png)

![image.png](attachment:9eaf9726-d28a-4f98-b820-aa5ca73f99ac:image.png)

![image.png](attachment:2c42383a-b8d2-4347-ab72-213bb11ad740:image.png)

## 9. DHCP Services

Dynamic IP addressing was configured on the router to automate host configuration.

### Result

- PCs automatically received IP addresses per VLAN
- Reduced manual configuration errors

### Validation

- DHCP lease confirmation on client machines

![Creating DHCP pool on Router](attachment:89e00de7-a8f1-4c20-becc-f9f70bb74e84:image.png)

Creating DHCP pool on Router

#### Testing DHCP on PCs

![SW3-HR-03](attachment:a5ba4eab-ec38-4e11-aa76-675864816e4b:image.png)

SW3-HR-03

![SW1-SALES-02](attachment:6b2ad708-5204-40b8-bcb2-28f4490aa390:image.png)

SW1-SALES-02

## 10. Management VLAN (VLAN 99)

A dedicated management VLAN was created for secure remote administration of network devices.

### Purpose

- Isolates management traffic
- Enables secure device administration

On SW-DIST-1

`interface vlan 99
ip address 192.168.99.1 255.255.255.0
no shutdown`

![image.png](attachment:7391d351-7d76-4cc6-833e-844cbc76abde:image.png)

On Access Switches

`interface vlan 99
ip address 192.168.99.2 255.255.255.0
no shutdown`

`ip default-gateway 192.168.99.1`

| Switch | IP |
| --- | --- |
| SW-DIST-1 | 192.168.99.1 |
| SW-ACCESS-1 | 192.168.99.2 |
| SW-ACCESS-2 | 192.168.99.3 |
| SW-ACCESS-3 | 192.168.99.4 |

![Switch Access 1](attachment:ffd79de6-183d-431d-92bc-6683b8d24775:image.png)

Switch Access 1

![Switch Access 2](attachment:fc2e7b3c-8d5d-4987-9175-43bf1fd2b7d5:image.png)

Switch Access 2

![Switch Access 3](attachment:249e2999-2d34-4dfd-827f-fb4ae28f76fd:image.png)

Switch Access 3

Why this is important

Now you can:

1. Ping switches from the end devices
2. Manage them remotely

#### Verify

`ping 192.168.99.1`

![SW-ACCESS-1 ping SW-DIST-1](attachment:0423f283-4a2a-4929-8bed-4fa9c0bff828:image.png)

SW-ACCESS-1 ping SW-DIST-1

![SW-ACCESS-3 ping SW-ACCESS-1](attachment:36c6d023-9316-4286-a332-ec856d565915:image.png)

SW-ACCESS-3 ping SW-ACCESS-1

Ran `show ip arp` to view the learned MAC addresses and got this result.

![Switch Access 1](attachment:ed43d417-a6c5-4160-a381-445308b67b52:image.png)

Switch Access 1

## 11. Port Security Implementation

Port security was implemented to prevent unauthorized device access. Ensured that all used access ports were configured to allow only one device. Once the port has been assigned and has learned the MAC address, it will not allow any other device to connect.

### Features Enabled

- Sticky MAC address learning
- Violation mode: restrict
- Single host per access port

### Outcome

- Unauthorized devices are blocked
- MAC addresses are dynamically learned and locked

![SW-ACCESS-1](attachment:d35ed9d4-e481-4e0a-bc2b-955fd4fa273f:image.png)

SW-ACCESS-1

![SW-ACCESS-2](attachment:af5a0559-1229-4bd7-bf18-a33e5cb0fdb0:image.png)

SW-ACCESS-2

Also did this with switch 3

![Before learning the mac address of the pc connected to fastEthernet0/1 on SW-ACCESS-3](attachment:3ba9b2ad-ca9c-4831-acb8-1765aee211e5:image.png)

Before learning the mac address of the pc connected to fastEthernet0/1 on SW-ACCESS-3

I sent a ping request from the pc connected to fastEthernet0/1 and then ran the show port-security interface fastEthernet0/1 to see the sticky mac address and got this result from the cli on SW-ACCESS-3

![ping request from the pc connected to fastEthernet0/1 on SW-ACCESS-3](attachment:be971431-65e7-4a71-b948-c9b232e15dd9:42319f02-3013-4872-aa7c-3003d0c9e657.png)

ping request from the pc connected to fastEthernet0/1 on SW-ACCESS-3

![After learning the mac address of the pc connected to fastEthernet0/1 on SW-ACCESS-3](attachment:6b142c87-c48b-4514-9e09-71aedea6cc58:image.png)

After learning the mac address of the pc connected to fastEthernet0/1 on SW-ACCESS-3

### Verify ping request

![Ping request from SW1-SALES-01 to SW2-IT-02](attachment:ca4b3b61-8fe6-4faf-834b-a293cbb0e61f:image.png)

Ping request from SW1-SALES-01 to SW2-IT-02

![show port-security interface fastEthernet0/1](attachment:aee312e6-8b87-43d3-a663-fcf7b6f2f044:image.png)

show port-security interface fastEthernet0/1

## 12. Connectivity Testing & Validation

### Test Cases

| Test | Result |
| --- | --- |
| Inter-VLAN Ping | Successful |
| DHCP Assignment | Successful |
| Switch-to-Switch Connectivity | Successful |
| Management VLAN Access | Successful |

### Observations

- All VLANs communicate via router successfully
- STP correctly blocks redundant links
- Port security actively restricts unauthorized access

SHOW RUNNING-CONFIG result on SW-ACCESS-1

![show running-config on SW-ACCESS-1](attachment:8eeeaaac-6804-471e-bf84-08580621aac1:image.png)

show running-config on SW-ACCESS-1

![show running-config on SW-ACCESS-1](attachment:98da43f2-468c-4549-a708-d6abb215ec1c:image.png)

show running-config on SW-ACCESS-1

![show running-config on SW-ACCESS-1](attachment:21c477ea-5f9c-4392-8d29-62f9eb676f66:image.png)

show running-config on SW-ACCESS-1

Improvements

- SSH configuration for secure remote access
- Access Control Lists (ACLs) for traffic filtering
- Network redundancy improvements (HSRP/VRRP)
- Network monitoring tools integration
