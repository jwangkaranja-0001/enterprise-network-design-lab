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

<img width="1365" height="767" alt="image" src="https://github.com/user-attachments/assets/e08fb70c-274e-45d3-9370-07a2eff2dda8" />

### Design Principles

- Hierarchical network design (Access → Distribution → Core)
- VLAN segmentation per department
- Centralized routing at the core router
- Redundant trunk links for resilience

## 3. VLAN & IP Addressing Design

### VLAN Structure

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

## VLAN Implementation

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
<img width="1365" height="767" alt="image" src="https://github.com/user-attachments/assets/561db099-8943-4893-91a3-7c7f9f40dbdf" />

Switch Distribution 1

<img width="1365" height="767" alt="image" src="https://github.com/user-attachments/assets/b9e362cd-5ae3-4291-a656-b23d1d63260b" />


Switch Access 2

<img width="1365" height="767" alt="image" src="https://github.com/user-attachments/assets/abd919f1-02af-408f-992a-4ec1dc921035" />

Switch Access 3

## Access Port Assignment

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

<img width="1365" height="767" alt="image" src="https://github.com/user-attachments/assets/773f9e06-ae3a-41ef-b116-d90fcd8ee0df" />


Switch Access 1

<img width="1365" height="767" alt="image" src="https://github.com/user-attachments/assets/c9291d86-a837-4d28-ad41-6d097b09ab9b" />


Switch Access 2




Switch access 3

## Trunking Configuration

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

<img width="1365" height="767" alt="image" src="https://github.com/user-attachments/assets/14b5635f-fa25-423c-97f1-dd22cc77aa6b" />


Switch Access Two - Configure trunks

<img width="1365" height="767" alt="image" src="https://github.com/user-attachments/assets/0185c932-e58d-4e26-95ef-001d580251fa" />


Switch Access 1 - Verify Trunks

<img width="1365" height="767" alt="image" src="https://github.com/user-attachments/assets/8f61260c-582d-4c61-a089-46432291ef79" />


Switch Access 3 - Configured Trunks and verified results

<img width="1365" height="767" alt="image" src="https://github.com/user-attachments/assets/24ba996d-9cba-4a48-abee-0b76bd019f76" />


Switch Distribution 1 - Configured trunks

We have multiple trunks between access switches, and this creates redundancy, which means STP, Spanning tree will block one path. 

STP is doing

1. Detecting loops 
2. Blocking one link
3. Keeping backup path ready

### STP Behavior

Spanning Tree Protocol automatically prevents loops by blocking redundant paths while maintaining backup links.

## Spanning Tree Protocol (STP)

### Configuration

SW-DIST-1 was configured as the root bridge:

`spanning-tree vlan 10,20,30,99 root primary`

### Role of STP

- Prevents Layer 2 switching loops
- Ensures single active path per VLAN
- Maintains backup redundant links

### Result

SW-DIST-1 acts as the central switching decision point for the campus network.

<img width="916" height="767" alt="image" src="https://github.com/user-attachments/assets/9fed93ea-bba1-420f-a466-1946637a5802" />


Verification results of: show spanning-tree before configuration

<img width="916" height="767" alt="image" src="https://github.com/user-attachments/assets/caef513f-9616-4cf1-9640-c785bffd4c6d" />

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

<img width="1365" height="767" alt="image" src="https://github.com/user-attachments/assets/8572cab8-8f0a-44c0-9a18-e4df33ff22a1" />


Trunking on SW-DIST-1

<img width="1365" height="767" alt="image" src="https://github.com/user-attachments/assets/ee359be5-7d0c-49e9-a59e-fad7537b8280" />

Trunking on R-CORE-1

<img width="1365" height="767" alt="image" src="https://github.com/user-attachments/assets/ac0ea983-dead-4e61-8913-251bccb038b9" />

Network after no shutdown on R-CORE-1

<img width="1363" height="767" alt="image" src="https://github.com/user-attachments/assets/9087606b-74a5-40fa-9065-9d015a5d760a" />


801.1Q Tagging on vlans to separate traffic

<img width="1365" height="767" alt="image" src="https://github.com/user-attachments/assets/b1ccc24a-003e-4daa-ba0e-9e492fd5a2f7" />


Tagging to separate traffic logically

### Testing Connectivity after manual configuration

Manually set ip addresses on 3 pcs and then ran ping across different vlans. 

Results showed ping results successful. 

<img width="755" height="767" alt="image" src="https://github.com/user-attachments/assets/4a45f34a-bbde-4c32-b652-148dfebe42fd" />

<img width="757" height="767" alt="image" src="https://github.com/user-attachments/assets/40f2c84e-cc50-4646-b9a5-14bd6e677c30" />


<img width="1365" height="767" alt="image" src="https://github.com/user-attachments/assets/06cc8f79-b970-4a2f-bcc5-90444a0d2312" />



## DHCP Services

Dynamic IP addressing was configured on the router to automate host configuration.

### Result

- PCs automatically received IP addresses per VLAN
- Reduced manual configuration errors

### Validation

- DHCP lease confirmation on client machines

<img width="1365" height="767" alt="image" src="https://github.com/user-attachments/assets/45481ff9-4230-4363-9a0f-18ac7d70a726" />

Creating DHCP pool on Router

#### Testing DHCP on PCs

<img width="1365" height="767" alt="image" src="https://github.com/user-attachments/assets/80322f6a-370a-4c51-80e7-c6b3bb0a4569" />


SW3-HR-03

<img width="1365" height="767" alt="image" src="https://github.com/user-attachments/assets/8ad0d969-9bdb-49d6-9aca-24f49f51126a" />


SW1-SALES-02

## Management VLAN (VLAN 99)

A dedicated management VLAN was created for secure remote administration of network devices.

### Purpose
<img width="630" height="185" alt="image" src="https://github.com/user-attachments/assets/432452d1-ef53-4c60-9d88-4e07e1e708f7" />

- Isolates management traffic
- Enables secure device administration

On SW-DIST-1

`interface vlan 99
ip address 192.168.99.1 255.255.255.0
no shutdown`


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

<img width="728" height="253" alt="image" src="https://github.com/user-attachments/assets/42642e8a-ff97-41b8-8133-4459b7bc9062" />


Switch Access 1

<img width="723" height="320" alt="image" src="https://github.com/user-attachments/assets/6a943062-a1c0-437e-9c03-135eccdf6cc4" />


Switch Access 2

<img width="728" height="246" alt="image" src="https://github.com/user-attachments/assets/da53c93a-4b0f-460e-b469-d06d321ed417" />


Switch Access 3

Why this is important

Now you can:

1. Ping switches from the end devices
2. Manage them remotely

#### Verify

`ping 192.168.99.1`

<img width="745" height="234" alt="image" src="https://github.com/user-attachments/assets/c1d50700-8b96-4f40-82ca-b8aad1e9bf79" />


SW-ACCESS-1 ping SW-DIST-1

<img width="748" height="240" alt="image" src="https://github.com/user-attachments/assets/51649889-ad2c-4990-9b8b-d9849854b159" />


SW-ACCESS-3 ping SW-ACCESS-1

Ran `show ip arp` to view the learned MAC addresses and got this result.

<img width="768" height="322" alt="image" src="https://github.com/user-attachments/assets/30442f26-f440-42fb-ab4c-0b0197fa8c63" />


Switch Access 1

## Port Security Implementation

Port security was implemented to prevent unauthorized device access. Ensured that all used access ports were configured to allow only one device. Once the port has been assigned and has learned the MAC address, it will not allow any other device to connect.

### Features Enabled

- Sticky MAC address learning
- Violation mode: restrict
- Single host per access port

### Outcome

- Unauthorized devices are blocked
- MAC addresses are dynamically learned and locked

<img width="656" height="156" alt="image" src="https://github.com/user-attachments/assets/000f1a46-1869-4f44-a00f-f5bc805f5f3e" />


SW-ACCESS-1

<img width="697" height="162" alt="image" src="https://github.com/user-attachments/assets/c0c0ba62-4282-467d-9fb6-667e6f3f8b83" />

SW-ACCESS-2

Also did this with switch 3

<img width="606" height="191" alt="image" src="https://github.com/user-attachments/assets/700e1036-aed2-4caf-a207-23b1d62edbc8" />


Before learning the mac address of the pc connected to fastEthernet0/1 on SW-ACCESS-3

I sent a ping request from the pc connected to fastEthernet0/1 and then ran the show port-security interface fastEthernet0/1 to see the sticky mac address and got this result from the cli on SW-ACCESS-3

<img width="754" height="321" alt="image" src="https://github.com/user-attachments/assets/8570ab71-27db-4bb2-a99f-eb68215b5fed" />


ping request from the pc connected to fastEthernet0/1 on SW-ACCESS-3

<img width="654" height="183" alt="image" src="https://github.com/user-attachments/assets/154eedd2-a2c0-4c22-a237-aa52598f4faf" />


After learning the mac address of the pc connected to fastEthernet0/1 on SW-ACCESS-3

### Verify ping request

<img width="759" height="386" alt="image" src="https://github.com/user-attachments/assets/799066ba-d74e-4a3e-af3a-edde378341a6" />


Ping request from SW1-SALES-01 to SW2-IT-02

<img width="569" height="229" alt="image" src="https://github.com/user-attachments/assets/bbd3c583-0c01-45ec-b2c1-46e631c559ef" />


show port-security interface fastEthernet0/1

## Connectivity Testing & Validation

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

<img width="540" height="767" alt="image" src="https://github.com/user-attachments/assets/7cbb5812-cbd1-48a9-8222-feacf1c450c7" />


show running-config on SW-ACCESS-1

<img width="601" height="767" alt="image" src="https://github.com/user-attachments/assets/c4f12792-a50c-4c3a-978f-6d99f57700a8" />

show running-config on SW-ACCESS-1

<img width="604" height="767" alt="image" src="https://github.com/user-attachments/assets/b2d6e465-eaa3-4597-975a-0415fe84b95c" />


show running-config on SW-ACCESS-1

## Improvements

- SSH configuration for secure remote access
- Access Control Lists (ACLs) for traffic filtering
- Network redundancy improvements (HSRP/VRRP)
- Network monitoring tools integration
