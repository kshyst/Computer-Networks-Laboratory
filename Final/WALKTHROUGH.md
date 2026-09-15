# Complete Guide to the Network Laboratory Final Exam

This guide is written for building the final scenario in **Cisco Packet Tracer**. All commands are without a Prompt, and each block is executed only on the device named immediately above it.

## Official Exam Deliverables

According to the first page of the question sheet:

- The exam is individual, and the project must be built independently.
- Only the Packet Tracer simulation file with the `.pkt` extension and a video of the project execution are submitted.
- Preparing a written Word or PDF report is not required.
- At the beginning of the video, briefly introduce the scenario structure and then run all mandatory tests in order.
- The recommended video duration is a maximum of 10 minutes.
- The deadline stated in the question sheet is the end of Saturday, 13 Tir.

The PNG Checkpoints in this guide are only for personal checking and video preparation and are not part of the official submission files. Any Checkpoint containing multiple devices or multiple windows can be divided into several readable images using the suffixes `-a`, `-b` and `-c`; the base name and the count of 32 Checkpoints do not change.

---

# 1. Practical Interpretation of Two Addressing Contradictions in the Question Sheet

Two mask values in the PDF are inconsistent with the other requirements. The solution below is a **practical interpretation for Packet Tracer**, not a literal implementation of those two masks; a literal implementation makes the mandatory tests impossible.

The question sheet uses the following addresses:

- `ISP ↔ R-EDGE` link: addresses `203.0.113.1` and `203.0.113.2`
- Public PAT address: `203.0.113.10`
- Public web address: `203.0.113.20`
- Internet user: `203.0.113.50/24`

If the ISP link is built with a `/30` mask, addresses `.10`, `.20` and `.50` will be outside that subnet, and the mandatory NAT tests in Packet Tracer will not work directly. To preserve all mandatory IPs and actually run the tests, the external segment in this guide is a `/24` network:

```text
203.0.113.0/24
```

Also, using `10.10.0.1/16` on the `R-EDGE ↔ SW-CORE` link creates an overlap with the `10.10.x.0/28` SVIs on SW-CORE. The routing link is built as follows while preserving the given R-EDGE IP:

```text
R-EDGE Gi0/0/1 = 10.10.0.1/30
SW-CORE Gi0/1 = 10.10.0.2/30
```

These two corrections only make the link masks consistent; all VLAN, server, NAT and Internet user addresses remain exactly as specified in the question sheet.

---

# 2. Required Equipment

Place the following equipment in an empty project:

- Two **Cisco ISR 4321** routers named `R-EDGE` and `ISP`
- One **3560-24PS** switch named `SW-CORE`
- Four **2960-24TT** switches named `SW-F1`, `SW-F2`, `SW-F3` and `INTERNET-SW`
- Eighteen internal PCs, two PCs for each team
- One PC named `Internet-User`
- Five Server-PTs named `WEB1`, `WEB2`, `TEST`, `DNS` and `DNS-ISP`

PC names:

| Floor | VLAN | First PC | Second PC |
|---|---:|---|---|
| Floor 1 | 10 | `PC-S1` | `PC-S2` |
| Floor 1 | 11 | `PC-M1` | `PC-M2` |
| Floor 1 | 12 | `PC-U1` | `PC-U2` |
| Floor 2 | 20 | `PC-B1` | `PC-B2` |
| Floor 2 | 21 | `PC-F1` | `PC-F2` |
| Floor 2 | 22 | `PC-Q1` | `PC-Q2` |
| Floor 3 | 30 | `PC-FN1` | `PC-FN2` |
| Floor 3 | 31 | `PC-H1` | `PC-H2` |
| Floor 3 | 32 | `PC-N1` | `PC-N2` |

## Cable Connections

Use **Automatically Choose Connection Type**. The `INTERNET-SW` switch only serves as the multi-access Cloud/ISP segment of the diagram in Packet Tracer and is not part of the company Campus.

| Source | Port | Destination | Port |
|---|---|---|---|
| R-EDGE | `Gi0/0/0` | INTERNET-SW | `Gi0/1` |
| ISP | `Gi0/0/0` | INTERNET-SW | `Gi0/2` |
| Internet-User | `Fa0` | INTERNET-SW | `Fa0/1` |
| DNS-ISP | `Fa0` | INTERNET-SW | `Fa0/2` |
| R-EDGE | `Gi0/0/1` | SW-CORE | `Gi0/1` |
| SW-CORE | `Fa0/21` | SW-F1 | `Gi0/1` |
| SW-CORE | `Fa0/22` | SW-F2 | `Gi0/1` |
| SW-CORE | `Fa0/23` | SW-F3 | `Gi0/1` |
| WEB1 | `Fa0` | SW-CORE | `Fa0/1` |
| WEB2 | `Fa0` | SW-CORE | `Fa0/2` |
| TEST | `Fa0` | SW-CORE | `Fa0/3` |
| DNS | `Fa0` | SW-CORE | `Fa0/4` |

On each Access Switch, connect the six PCs in order to `Fa0/1` through `Fa0/6`.

**Checkpoint `F-01-topology.png`:** Take a picture of the entire topology; the names of all devices and the cables between the Edge, Core and three Access Switches must be readable.

---

# 3. Final Addressing Table

| VLAN | Name | Network | Gateway | Client Method |
|---:|---|---|---|---|
| 10 | Sales | `10.10.10.0/28` | `10.10.10.1` | DHCP |
| 11 | Marketing | `10.10.11.0/28` | `10.10.11.1` | DHCP |
| 12 | Support | `10.10.12.0/28` | `10.10.12.1` | DHCP |
| 20 | Backend | `10.10.20.0/28` | `10.10.20.1` | DHCP |
| 21 | Frontend | `10.10.21.0/28` | `10.10.21.1` | DHCP |
| 22 | QA | `10.10.22.0/28` | `10.10.22.1` | DHCP |
| 30 | Finance | `10.10.30.0/28` | `10.10.30.1` | DHCP |
| 31 | HR | `10.10.31.0/28` | `10.10.31.1` | DHCP |
| 32 | Network | `10.10.32.0/28` | `10.10.32.1` | Static for the two network team members |
| 99 | Management | `10.10.99.0/28` | `10.10.99.1` | Static for switches |
| 100 | Server Farm | `10.10.100.0/28` | `10.10.100.1` | Static |
| 999 | Native | No IP | No Gateway | Trunk only |

Static addresses:

| Device | IP | Mask | Gateway | DNS |
|---|---|---|---|---|
| WEB1 | `10.10.100.10` | `255.255.255.240` | `10.10.100.1` | `10.10.100.13` |
| WEB2 | `10.10.100.11` | `255.255.255.240` | `10.10.100.1` | `10.10.100.13` |
| TEST | `10.10.100.12` | `255.255.255.240` | `10.10.100.1` | `10.10.100.13` |
| DNS | `10.10.100.13` | `255.255.255.240` | `10.10.100.1` | `10.10.100.13` |
| PC-N1 | `10.10.32.10` | `255.255.255.240` | `10.10.32.1` | `10.10.100.13` |
| PC-N2 | `10.10.32.11` | `255.255.255.240` | `10.10.32.1` | `10.10.100.13` |
| Internet-User | `203.0.113.50` | `255.255.255.0` | `203.0.113.1` | `203.0.113.53` |
| DNS-ISP | `203.0.113.53` | `255.255.255.0` | `203.0.113.1` | `203.0.113.53` |

---

# 4. Configuring ISP and R-EDGE

## Device: ISP

```text
enable
configure terminal
hostname ISP
interface GigabitEthernet0/0/0
 ip address 203.0.113.1 255.255.255.0
 no shutdown
exit
ip route 10.10.0.0 255.255.0.0 203.0.113.2
end
write memory
```

The `10.10.0.0/16` route is only there so that the mandatory Internet-User access test to TEST actually reaches the external ACL on R-EDGE and increments the deny counter.

## Device: R-EDGE

```text
enable
configure terminal
hostname R-EDGE
interface GigabitEthernet0/0/0
 description OUTSIDE-TO-ISP
 ip address 203.0.113.2 255.255.255.0
 ip nat outside
 no shutdown
exit
interface GigabitEthernet0/0/1
 description INSIDE-TO-SW-CORE
 ip address 10.10.0.1 255.255.255.252
 ip nat inside
 no shutdown
exit
ip route 0.0.0.0 0.0.0.0 203.0.113.1
end
write memory
```

## Device: R-EDGE — Initial Check

```text
enable
show ip interface brief
show ip route
```

Both interfaces are expected to be `up/up`, and the default route to `203.0.113.1` should be visible.

**Checkpoint `F-02-edge-isp-links.png`:** Record the `show ip interface brief` output and the R-EDGE default route.

---

# 5. Creating VLANs, SVIs and Trunks

## Device: SW-CORE

```text
enable
configure terminal
hostname SW-CORE
ip routing
vlan 10
 name Sales
vlan 11
 name Marketing
vlan 12
 name Support
vlan 20
 name Backend
vlan 21
 name Frontend
vlan 22
 name QA
vlan 30
 name Finance
vlan 31
 name HR
vlan 32
 name Network
vlan 99
 name Management
vlan 100
 name Server-Farm
vlan 999
 name Native
interface range FastEthernet0/1 - 4
 switchport mode access
 switchport access vlan 100
 spanning-tree portfast
 no shutdown
exit
interface FastEthernet0/21
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,11,12,99,999
 no shutdown
exit
interface FastEthernet0/22
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 20,21,22,99,999
 no shutdown
exit
interface FastEthernet0/23
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 30,31,32,99,999
 no shutdown
exit
interface Vlan10
 ip address 10.10.10.1 255.255.255.240
 no shutdown
exit
interface Vlan11
 ip address 10.10.11.1 255.255.255.240
 no shutdown
exit
interface Vlan12
 ip address 10.10.12.1 255.255.255.240
 no shutdown
exit
interface Vlan20
 ip address 10.10.20.1 255.255.255.240
 no shutdown
exit
interface Vlan21
 ip address 10.10.21.1 255.255.255.240
 no shutdown
exit
interface Vlan22
 ip address 10.10.22.1 255.255.255.240
 no shutdown
exit
interface Vlan30
 ip address 10.10.30.1 255.255.255.240
 no shutdown
exit
interface Vlan31
 ip address 10.10.31.1 255.255.255.240
 no shutdown
exit
interface Vlan32
 ip address 10.10.32.1 255.255.255.240
 no shutdown
exit
interface Vlan99
 ip address 10.10.99.1 255.255.255.240
 no shutdown
exit
interface Vlan100
 ip address 10.10.100.1 255.255.255.240
 no shutdown
exit
interface GigabitEthernet0/1
 no switchport
 ip address 10.10.0.2 255.255.255.252
 no shutdown
exit
end
write memory
```

## Device: SW-F1

```text
enable
configure terminal
hostname SW-F1
vlan 10
 name Sales
vlan 11
 name Marketing
vlan 12
 name Support
vlan 99
 name Management
vlan 999
 name Native
interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
exit
interface range FastEthernet0/3 - 4
 switchport mode access
 switchport access vlan 11
 spanning-tree portfast
exit
interface range FastEthernet0/5 - 6
 switchport mode access
 switchport access vlan 12
 spanning-tree portfast
exit
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,11,12,99,999
 no shutdown
exit
interface Vlan99
 ip address 10.10.99.11 255.255.255.240
 no shutdown
exit
ip default-gateway 10.10.99.1
end
write memory
```

## Device: SW-F2

```text
enable
configure terminal
hostname SW-F2
vlan 20
 name Backend
vlan 21
 name Frontend
vlan 22
 name QA
vlan 99
 name Management
vlan 999
 name Native
interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
exit
interface range FastEthernet0/3 - 4
 switchport mode access
 switchport access vlan 21
 spanning-tree portfast
exit
interface range FastEthernet0/5 - 6
 switchport mode access
 switchport access vlan 22
 spanning-tree portfast
exit
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 20,21,22,99,999
 no shutdown
exit
interface Vlan99
 ip address 10.10.99.12 255.255.255.240
 no shutdown
exit
ip default-gateway 10.10.99.1
end
write memory
```

## Device: SW-F3

```text
enable
configure terminal
hostname SW-F3
vlan 30
 name Finance
vlan 31
 name HR
vlan 32
 name Network
vlan 99
 name Management
vlan 999
 name Native
interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
exit
interface range FastEthernet0/3 - 4
 switchport mode access
 switchport access vlan 31
 spanning-tree portfast
exit
interface range FastEthernet0/5 - 6
 switchport mode access
 switchport access vlan 32
 spanning-tree portfast
exit
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 30,31,32,99,999
 no shutdown
exit
interface Vlan99
 ip address 10.10.99.13 255.255.255.240
 no shutdown
exit
ip default-gateway 10.10.99.1
end
write memory
```

## Device: SW-CORE — Checking VLAN and Trunk

```text
enable
show vlan brief
show interfaces trunk
show ip interface brief
```

**Checkpoint `F-03-core-vlans.png`:** Show all VLANs 10, 11, 12, 20, 21, 22, 30, 31, 32, 99, 100 and 999.

**Checkpoint `F-04-core-trunks.png`:** Show three active Trunks and a Native VLAN of 999.

**Checkpoint `F-05-core-svis.png`:** Record all SVIs and the routed interface connected to R-EDGE in the output.

---

# 6. DHCP on SW-CORE

## Device: SW-CORE

```text
enable
configure terminal
ip dhcp excluded-address 10.10.10.1
ip dhcp excluded-address 10.10.11.1
ip dhcp excluded-address 10.10.12.1
ip dhcp excluded-address 10.10.20.1
ip dhcp excluded-address 10.10.21.1
ip dhcp excluded-address 10.10.22.1
ip dhcp excluded-address 10.10.30.1
ip dhcp excluded-address 10.10.31.1
ip dhcp excluded-address 10.10.32.1 10.10.32.11
ip dhcp excluded-address 10.10.100.1 10.10.100.13
ip dhcp pool SALES
 network 10.10.10.0 255.255.255.240
 default-router 10.10.10.1
 dns-server 10.10.100.13
 lease 1
exit
ip dhcp pool MARKETING
 network 10.10.11.0 255.255.255.240
 default-router 10.10.11.1
 dns-server 10.10.100.13
 lease 1
exit
ip dhcp pool SUPPORT
 network 10.10.12.0 255.255.255.240
 default-router 10.10.12.1
 dns-server 10.10.100.13
 lease 1
exit
ip dhcp pool BACKEND
 network 10.10.20.0 255.255.255.240
 default-router 10.10.20.1
 dns-server 10.10.100.13
 lease 1
exit
ip dhcp pool FRONTEND
 network 10.10.21.0 255.255.255.240
 default-router 10.10.21.1
 dns-server 10.10.100.13
 lease 1
exit
ip dhcp pool QA
 network 10.10.22.0 255.255.255.240
 default-router 10.10.22.1
 dns-server 10.10.100.13
 lease 1
exit
ip dhcp pool FINANCE
 network 10.10.30.0 255.255.255.240
 default-router 10.10.30.1
 dns-server 10.10.100.13
 lease 1
exit
ip dhcp pool HR
 network 10.10.31.0 255.255.255.240
 default-router 10.10.31.1
 dns-server 10.10.100.13
 lease 1
exit
ip dhcp pool NETWORK
 network 10.10.32.0 255.255.255.240
 default-router 10.10.32.1
 dns-server 10.10.100.13
 lease 1
exit
ip dhcp pool SERVER-FARM
 network 10.10.100.0 255.255.255.240
 default-router 10.10.100.1
 dns-server 10.10.100.13
 lease 1
exit
end
write memory
```

The Pool for the Server Farm is created to comply with the phrase “one Pool for each VLAN”, but addresses `.1` through `.13` are excluded from it, and the four servers must remain Static. No Pool is created for the management VLAN because this VLAN is optional and is used only by equipment with static addresses.

For each VLAN, first set the first PC in the naming table to DHCP, and activate the second PC in the same VLAN only after an address has been received. In a fresh project, this order causes the first PC to receive address `.2` and the second PC to receive address `.3`. For all PCs in VLANs 10, 11, 12, 20, 21, 22, 30 and 31:

1. `Desktop > IP Configuration`
2. Select the **DHCP** option.
3. Wait until the IP, Gateway and DNS are displayed.

In VLAN 22, make sure to set `PC-Q1` to DHCP before `PC-Q2`, and confirm right there at the Checkpoint that `PC-Q1` has received address `10.10.22.2`. If the project already has Leases, create a fresh project or clear the previous Leases before continuing; the tests below are written based on this definite order.

For `PC-N1` and `PC-N2`, use the static address table in Section 3.

## Device: SW-CORE — Checking DHCP

```text
enable
show ip dhcp binding
show ip dhcp pool
```

**Checkpoint `F-06-dhcp-bindings.png`:** Record the Leases of the different VLANs and their MAC addresses.

**Checkpoint `F-07-two-client-ipconfig.png`:** On `PC-S1` and `PC-Q1`, run the `ipconfig` command and record the IP, Gateway and DNS. For readability, split the base name using the suffixes `-a` for PC-S1 and `-b` for PC-Q1.

---

# 7. Server Addressing and Services

On each server, go to `Desktop > IP Configuration` and enter the values from the table in Section 3.

## WEB1

1. `Services > HTTP`
2. Turn HTTP **On**.
3. Edit the `index.html` file so that the phrase `NetLab WEB1 Production` is clear.

## WEB2

1. `Services > HTTP`
2. Turn HTTP **On**.
3. Edit the `index.html` file so that the phrase `NetLab WEB2 Backup Portal` is clear.

## TEST

1. `Services > HTTP`
2. Turn HTTP **On**.
3. Edit the `index.html` file so that the phrase `NetLab Internal TEST Server` is clear.

**Checkpoint `F-08-server-addresses.png`:** Record the static IPs of all four internal servers in two or more views; `.10` through `.13` and Gateway `.1` must be readable.

---

# 8. Internal and External DNS

## Device: DNS

1. Go to `Services > DNS`.
2. Turn the DNS service **On**.
3. Add the following records one by one with the type `A Record`:

| Name | Address |
|---|---|
| `netlab.ir` | `203.0.113.20` |
| `www.netlab.ir` | `10.10.100.10` |
| `backup.netlab.ir` | `10.10.100.11` |
| `test.netlab.ir` | `10.10.100.12` |
| `ns.netlab.ir` | `10.10.100.13` |

These records create the internal Split DNS; `www` and `backup` go directly to private IPs from inside and do not depend on Hairpin NAT.

## Device: DNS-ISP

1. In `Desktop > IP Configuration`, set the address to `203.0.113.53/24`, the Gateway to `203.0.113.1` and the DNS to itself.
2. Go to `Services > DNS` and turn DNS **On**.
3. Add the following records:

| Name | Address |
|---|---|
| `netlab.ir` | `203.0.113.20` |
| `www.netlab.ir` | `203.0.113.20` |
| `backup.netlab.ir` | `203.0.113.20` |
| `www.techcorp.ir` | `203.0.113.20` |
| `backup.techcorp.ir` | `203.0.113.20` |

The question sheet uses `netlab.ir` in the DNS table, but one NAT clause mentions the name `techcorp.ir`. Defining both names resolves this textual contradiction without changing the public destination. Do not create any external record for TEST.

## Device: Internet-User

Enter the following in `Desktop > IP Configuration`:

```text
IP Address: 203.0.113.50
Subnet Mask: 255.255.255.0
Default Gateway: 203.0.113.1
DNS Server: 203.0.113.53
```

**Checkpoint `F-09-internal-dns-records.png`:** Record the five internal DNS records.

**Checkpoint `F-10-external-dns-records.png`:** Record the public records and the absence of a TEST record.

---

# 9. OSPF Area 0

## Device: SW-CORE

```text
enable
configure terminal
router ospf 1
 router-id 1.1.1.1
 passive-interface default
 no passive-interface GigabitEthernet0/1
 network 10.10.0.0 0.0.255.255 area 0
exit
end
write memory
```

## Device: R-EDGE

```text
enable
configure terminal
router ospf 1
 router-id 2.2.2.2
 passive-interface default
 no passive-interface GigabitEthernet0/0/1
 network 10.10.0.0 0.0.0.3 area 0
 default-information originate
exit
end
write memory
```

## Device: SW-CORE — Checking OSPF

```text
enable
show ip ospf neighbor
show ip route
```

Neighbor `2.2.2.2` should be visible in the `FULL` state, along with the OSPF default route marked `O*E2`.

## Device: R-EDGE — Checking Internal Routes

```text
enable
show ip ospf neighbor
show ip route ospf
```

The internal VLAN routes should be visible with the letter `O`.

**Checkpoint `F-11-ospf-neighbor.png`:** Record the FULL adjacency on SW-CORE.

**Checkpoint `F-12-core-routing-table.png`:** Record the default route and connected networks.

---

# 10. PAT and Port Forwarding on R-EDGE

## Device: R-EDGE

```text
enable
configure terminal
ip access-list extended PAT-USERS
 permit ip 10.10.10.0 0.0.0.15 any
 permit ip 10.10.11.0 0.0.0.15 any
 permit ip 10.10.12.0 0.0.0.15 any
 permit ip 10.10.20.0 0.0.0.15 any
 permit ip 10.10.21.0 0.0.0.15 any
 permit ip 10.10.22.0 0.0.0.15 any
 permit tcp 10.10.30.0 0.0.0.15 any eq www
 permit udp 10.10.30.0 0.0.0.15 any eq domain
 permit tcp 10.10.30.0 0.0.0.15 any eq domain
 permit ip 10.10.31.0 0.0.0.15 any
 permit ip 10.10.32.0 0.0.0.15 any
exit
ip nat pool EMPLOYEE-PAT 203.0.113.10 203.0.113.10 netmask 255.255.255.0
ip nat inside source list PAT-USERS pool EMPLOYEE-PAT overload
ip nat inside source static tcp 10.10.100.10 80 203.0.113.20 80
ip nat inside source static tcp 10.10.100.11 80 203.0.113.20 8080
end
write memory
```

The NAT ACL is defined for all user VLANs, not for the Server Farm, Management, or the Transit link. In the base configuration, Finance users can only send HTTP and DNS traffic to the internet through PAT; therefore, the “restricted access” policy is enforced even without the time-based bonus section.

Do not define any NAT or Port Forwarding for `10.10.100.12`.

## Device: PC-S1 — Generate PAT

```text
ping 203.0.113.1
```

## Device: R-EDGE — Check NAT

```text
enable
show ip nat translations
show ip nat statistics
```

A PAT translation with an Inside Local belonging to PC-S1 and an Inside Global of `203.0.113.10` should be visible. The two static TCP mappings should also always be present.

**Checkpoint `F-13-pat-translation.png`:** Capture the PAT entry immediately after the Ping.

**Checkpoint `F-14-static-port-forwarding.png`:** Capture the two mappings for WEB1 on port 80 and WEB2 on port 8080.

---

# 11. Internal Server ACL

The assignment states “ACL-SERVERS-IN on VLAN 100 as Inbound.” On an SVI, the `in` direction checks traffic that **enters SW-CORE from the servers**, whereas the requested rules must control user requests **destined for the servers**. To actually enforce the policy, the ACL is applied to `Vlan100` in the `out` direction. This choice does not match the direction stated in the PDF, but using `in` makes it impossible to carry out the QA, non-QA, and management tests with a destination-oriented ACL. State this contradiction briefly and explicitly in the video explanation.

## Device: SW-CORE

```text
enable
configure terminal
ip access-list extended ACL-SERVERS-IN
 permit udp any host 10.10.100.13 eq domain
 permit tcp any host 10.10.100.13 eq domain
 permit tcp any host 10.10.100.10 eq www
 permit tcp any host 10.10.100.11 eq www
 permit icmp 10.10.10.0 0.0.0.15 host 10.10.100.10 echo
 permit tcp 10.10.22.0 0.0.0.15 host 10.10.100.12 eq www
 permit ip 10.10.32.0 0.0.0.15 10.10.100.0 0.0.0.15
 deny tcp any 10.10.100.0 0.0.0.15 eq 22
 deny tcp any 10.10.100.0 0.0.0.15 eq telnet
 deny ip 10.10.30.0 0.0.0.15 10.10.100.0 0.0.0.15
 deny ip any host 10.10.100.12
 deny ip any 10.10.100.0 0.0.0.15
 permit ip any any
exit
interface Vlan100
 ip access-group ACL-SERVERS-IN out
exit
end
write memory
```

The ICMP rule from VLAN 10 to WEB1 has been added only to accommodate the mandatory Ping test on page 12 of the assignment. Other ordinary users receive only the permitted HTTP and DNS.

**Checkpoint `F-15-server-acl-config.png`:** Capture the ACL order and its attachment to `Vlan100 out`.

---

# 12. Internet Edge ACL

## Device: R-EDGE

```text
enable
configure terminal
ip access-list extended INTERNET-IN
 permit tcp any host 203.0.113.20 eq www
 permit tcp any host 203.0.113.20 eq 8080
 permit tcp any host 203.0.113.10 established
 permit udp any host 203.0.113.10 gt 1023
 permit icmp any host 203.0.113.10 echo-reply
 deny ip any 10.10.0.0 0.0.255.255
 deny ip any host 203.0.113.20
 deny ip any host 203.0.113.10
 deny ip any any
exit
interface GigabitEthernet0/0/0
 ip access-group INTERNET-IN in
exit
end
write memory
```

The first two Permit rules open only the public web. The next three Permit rules allow the responses needed for PAT connections initiated from inside. TEST has no public mapping, and direct traffic to the private network is also denied.

**Checkpoint `F-16-internet-acl-config.png`:** Capture the external ACL and its inbound attachment to `Gi0/0/0`.

---

# 13. SSH/Telnet Management for the Network Team Only

The standard Server-PT has no SSH/Telnet service that can be enabled in many versions of Packet Tracer, whereas the assignment requires a connection to “the same server” and the diagram shows WEB1 with HTTP+SSH. This is an unresolvable limitation of the Server-PT model and must not be presented in the video as a complete server implementation.

If the SSH option is available in `WEB1 > Services`, set it to On, create the user `netadmin` with the password `NetLab32Pass`, and perform the ordinary-user and Network Team tests directly with `10.10.100.10`. If this option is not available, perform the demonstration in two parts:

1. In Simulation Mode, use **Add Complex PDU** to create a TCP packet with Destination Port set to 22 from PC-S1 to WEB1; the packet must be dropped on SW-CORE by the ACL.
2. Create the same packet from PC-N1 to WEB1; the ACL must allow it through to WEB1, although Server-PT itself does not complete the application connection because it lacks an SSH service.
3. To demonstrate a real management session, compare the ordinary-user and Network Team SSH connections to the same destination, R-EDGE.

This method both demonstrates the ACL decision for the server destination and honestly distinguishes the simulator limitation from the success of the SSH session. The management access policy for network equipment is applied using the following blocks.

## Device: R-EDGE

```text
enable
configure terminal
ip domain-name netlab.ir
username netadmin privilege 15 secret NetLab32Pass
crypto key generate rsa general-keys modulus 1024
ip ssh version 2
ip access-list standard MGMT-VTY
 permit 10.10.32.0 0.0.0.15
 deny any
exit
line vty 0 4
 login local
 transport input ssh telnet
 access-class MGMT-VTY in
exit
end
write memory
```

## Device: SW-CORE

```text
enable
configure terminal
ip domain-name netlab.ir
username netadmin privilege 15 secret NetLab32Pass
crypto key generate rsa general-keys modulus 1024
ip ssh version 2
ip access-list standard MGMT-VTY
 permit 10.10.32.0 0.0.0.15
 deny any
exit
line vty 0 15
 login local
 transport input ssh telnet
 access-class MGMT-VTY in
exit
end
write memory
```

## Device: SW-F1

```text
enable
configure terminal
ip domain-name netlab.ir
username netadmin privilege 15 secret NetLab32Pass
crypto key generate rsa general-keys modulus 1024
ip ssh version 2
ip access-list standard MGMT-VTY
 permit 10.10.32.0 0.0.0.15
 deny any
exit
line vty 0 15
 login local
 transport input ssh telnet
 access-class MGMT-VTY in
exit
end
write memory
```

## Device: SW-F2

```text
enable
configure terminal
ip domain-name netlab.ir
username netadmin privilege 15 secret NetLab32Pass
crypto key generate rsa general-keys modulus 1024
ip ssh version 2
ip access-list standard MGMT-VTY
 permit 10.10.32.0 0.0.0.15
 deny any
exit
line vty 0 15
 login local
 transport input ssh telnet
 access-class MGMT-VTY in
exit
end
write memory
```

## Device: SW-F3

```text
enable
configure terminal
ip domain-name netlab.ir
username netadmin privilege 15 secret NetLab32Pass
crypto key generate rsa general-keys modulus 1024
ip ssh version 2
ip access-list standard MGMT-VTY
 permit 10.10.32.0 0.0.0.15
 deny any
exit
line vty 0 15
 login local
 transport input ssh telnet
 access-class MGMT-VTY in
exit
end
write memory
```

**Checkpoint `F-17-management-vty-acl.png`:** Capture the management ACL and VTY lines on R-EDGE.

---

# 14. Bonus Section: Finance Time Restriction

The assignment does not specify exact hours. As an example, Finance access is permitted on weekdays from 08:00 to 18:00. Before execution, synchronize the Packet Tracer clock with the test scenario.

## Device: R-EDGE

```text
enable
configure terminal
time-range FINANCE-HOURS
 periodic weekdays 08:00 to 18:00
exit
ip access-list extended PAT-USERS
 no permit tcp 10.10.30.0 0.0.0.15 any eq www
 no permit udp 10.10.30.0 0.0.0.15 any eq domain
 no permit tcp 10.10.30.0 0.0.0.15 any eq domain
 permit tcp 10.10.30.0 0.0.0.15 any eq www time-range FINANCE-HOURS
 permit udp 10.10.30.0 0.0.0.15 any eq domain time-range FINANCE-HOURS
 permit tcp 10.10.30.0 0.0.0.15 any eq domain time-range FINANCE-HOURS
exit
end
write memory
```

If the Packet Tracer version does not support the `time-range` command, do not implement this bonus section and retain the original PAT configuration.

---

# 15. Mandatory Tests in Video Order

## Test 1 — VLAN, Trunk, and SVI

### Device: SW-CORE

```text
enable
show vlan brief
show interfaces trunk
show ip interface brief
```

Items that must be readable in the video:

- All VLANs have been created.
- `Fa0/21`, `Fa0/22`, and `Fa0/23` are Trunk ports.
- The Native VLAN of all three links is `999`.
- The user and Server Farm SVIs have the correct addresses.

**Checkpoint `F-18-video-vlan-trunk-svi.png`**

## Test 2 — DHCP on Two Different VLANs

### Device: PC-S1

```text
ipconfig
```

Expected in a fresh project:

```text
IP: 10.10.10.2/28
Gateway: 10.10.10.1
DNS: 10.10.100.13
```

### Device: PC-Q1

```text
ipconfig
```

Expected in a fresh project:

```text
IP: 10.10.22.2/28
Gateway: 10.10.22.1
DNS: 10.10.100.13
```

**Checkpoint `F-19-video-dhcp-two-vlans.png`:** Capture the two outputs as `-a` for PC-S1 and `-b` for PC-Q1; the IP, Mask, Gateway, and DNS must be fully readable.

## Test 3 — Routing and OSPF

### Device: SW-CORE

```text
enable
show ip route
show ip ospf neighbor
```

### Device: PC-S1

```text
ping 10.10.22.2
ping 10.10.100.10
```

Both Pings must succeed. The second Ping is permitted by the specific ACL exception for the mandatory test.

**Checkpoint `F-20-video-ospf-and-pings.png`:** Use `-a` for Route/Neighbor on SW-CORE and `-b` for the two successful PC-S1 Pings.

## Test 4 — DNS and Internal Pages

### Device: PC-S1

```text
nslookup www.netlab.ir
nslookup backup.netlab.ir
```

Expected results:

```text
www.netlab.ir -> 10.10.100.10
backup.netlab.ir -> 10.10.100.11
```

Then open the following in `Desktop > Web Browser` on this same PC:

```text
http://www.netlab.ir
http://backup.netlab.ir
```

The WEB1 and WEB2 pages must open without using the IP.

**Checkpoint `F-21-video-internal-dns.png`**

**Checkpoint `F-22-video-internal-web-pages.png`**

## Test 5 — PAT

### Device: PC-S1

```text
ping 203.0.113.1
```

### Device: R-EDGE

```text
enable
show ip nat translations
show ip nat statistics
```

The PAT entry must have an Inside Global of `203.0.113.10`. Since the ICMP translation expires quickly, display the NAT table immediately.

**Checkpoint `F-23-video-pat.png`**

## Test 6 — Static NAT from Internet-User

In the Web Browser on `Internet-User`, first open the two URLs required by the assignment in order:

```text
http://www.techcorp.ir
http://backup.techcorp.ir:8080
```

Expected result:

- The first URL should open the `NetLab WEB1 Production` page.
- The second URL, with port 8080, should open the `NetLab WEB2 Backup Portal` page.

To demonstrate compatibility with the company’s primary domain name, these two names must also produce the same results:

```text
http://www.netlab.ir
http://backup.netlab.ir:8080
```

### Device: R-EDGE

```text
enable
show ip nat translations
```

**Checkpoint `F-24-video-public-web1.png`:** The WEB1 page and the address bar containing `http://www.techcorp.ir` must be readable simultaneously.

**Checkpoint `F-25-video-public-web2.png`:** The WEB2 page and the address bar containing `http://backup.techcorp.ir:8080` must be readable simultaneously.

**Checkpoint `F-26-video-static-nat-table.png`**

## Test 7 — QA Access to TEST Must Be Allowed

Open the following in the Web Browser on `PC-Q1`:

```text
http://test.netlab.ir
```

The `NetLab Internal TEST Server` page must open.

**Checkpoint `F-27-video-qa-test-allowed.png`**

## Test 8 — Non-QA User Access to TEST Must Be Blocked

Open the following in the Web Browser on `PC-S1`:

```text
http://test.netlab.ir
```

DNS must resolve the name to `10.10.100.12`, but the page must not open. This difference shows that the problem is not DNS and that the ACL has blocked access.

**Checkpoint `F-28-video-nonqa-test-blocked.png`**

## Test 9 — The Ordinary User’s SSH Path to WEB1 Must Be Denied

In **Simulation Mode**, create an **Add Complex PDU**:

```text
Source Device: PC-S1
Destination Device: WEB1
Protocol: TCP
Source Port: 1025
Destination Port: 22
One Shot
```

The packet must be dropped on SW-CORE by the ACL, and the SSH deny rule counter must increase. Then, to demonstrate that a real equipment management session is denied, run:

### Device: PC-S1

```text
ssh -l netadmin 10.10.0.1
```

The connection to R-EDGE must be denied or Timeout because of `MGMT-VTY`.

**Checkpoint `F-29-video-ordinary-ssh-denied.png`:** If needed, split the base name using the suffix `-a` for the packet Drop to WEB1 and `-b` for the denied R-EDGE session.

## Test 10 — The Network Team’s SSH Path to the Same WEB1 Must Be Allowed

In **Simulation Mode**, create the same Complex PDU, this time with PC-N1 as the source:

```text
Source Device: PC-N1
Destination Device: WEB1
Protocol: TCP
Source Port: 1025
Destination Port: 22
One Shot
```

The packet must pass through the ACL and reach WEB1. If the SSH service on WEB1 is available in your version, also perform the real test directly:

### Device: PC-N1

```text
ssh -l netadmin 10.10.100.10
```

If Server-PT has no SSH service, run the following from the same PC to demonstrate a real management session on the equipment:

### Device: PC-N1

```text
ssh -l netadmin 10.10.0.1
```

Test password:

```text
NetLab32Pass
```

The R-EDGE connection must be established. After seeing the device Prompt, leave with `exit`. Explicitly state in the video that the Network team’s PDU reaching WEB1 proves that the server ACL permits it, but completing the WEB1 application session depends on the presence of an SSH service in the Server-PT model.

**Checkpoint `F-30-video-network-ssh-allowed.png`:** If needed, split the base name using the suffix `-a` for the packet reaching WEB1 and `-b` for the successful R-EDGE session.

## Test 11 — Internet-User Access to TEST Must Be Blocked

### Device: Internet-User

```text
nslookup test.netlab.ir
ping 10.10.100.12
```

Expected result:

- `test.netlab.ir` has no record in the external DNS.
- A direct Ping to the private IP of TEST is also denied by the external ACL on R-EDGE.
- No public URL or Port Forwarding exists for TEST.

**Checkpoint `F-31-video-internet-test-blocked.png`**

## Test 12 — ACL Counters

### Device: SW-CORE

```text
enable
show access-lists ACL-SERVERS-IN
```

### Device: R-EDGE

```text
enable
show access-lists INTERNET-IN
show access-lists MGMT-VTY
```

The Permit and Deny rules used in the previous tests must have counters.

**Checkpoint `F-32-video-acl-counters.png`:** Capture the SW-CORE output in `-a` and the two R-EDGE ACLs in `-b`; the Permit and Deny Match counts must be readable.

---

# 16. Brief Troubleshooting

## If an SVI Is Down

- At least one Access or Trunk port carrying that VLAN must be `up`.
- The VLAN must exist on SW-CORE and the corresponding Access Switch.
- The Native VLAN on both sides of the Trunk must be `999`.

## If DHCP Does Not Work

### Device: SW-CORE

```text
enable
show ip dhcp pool
show ip dhcp binding
show ip interface brief
```

Check that the SVI of the relevant VLAN is `up/up` and that the Pool has a free address.

## If OSPF Has No Neighbor

### Device: SW-CORE

```text
enable
show ip ospf interface GigabitEthernet0/1
show ip ospf neighbor
```

### Device: R-EDGE

```text
enable
show ip ospf interface GigabitEthernet0/0/1
show ip ospf neighbor
```

Both sides must be in `10.10.0.0/30` and Area 0.

## If WEB2 Does Not Open from the Internet

- The URL must include port `8080`.
- The internal WEB2 mapping is still to port `80`.
- The external ACL must permit destination port `8080` on `203.0.113.20`.

## If the NAT Table Is Empty

First Ping `203.0.113.1` from an internal PC and immediately run the following on R-EDGE:

```text
enable
show ip nat translations
```

---

# 17. Suggested Sequence for a Video of at Most 10 Minutes

| Approximate Time | Display |
|---|---|
| 0:00 to 0:45 | Full view of the topology and Edge/Core/Access architecture |
| 0:45 to 1:45 | VLAN, Trunk, and SVI on SW-CORE |
| 1:45 to 2:30 | DHCP on two clients from two VLANs |
| 2:30 to 3:30 | OSPF, Route, and the two mandatory Pings |
| 3:30 to 4:30 | nslookup and internal pages by domain name |
| 4:30 to 5:30 | PAT and the translation table |
| 5:30 to 6:30 | WEB1 and WEB2 from Internet-User |
| 6:30 to 8:15 | QA allowed, non-QA blocked, external TEST blocked |
| 8:15 to 9:15 | Ordinary-user and Network Team SSH |
| 9:15 to 10:00 | ACL counters and summary |

---

# 18. Complete Final Checklist

## Topology and Layer 2

- [ ] R-EDGE and ISP are ISR 4321 models.
- [ ] SW-CORE is a 3560-24PS model.
- [ ] Each floor has an independent 2960.
- [ ] There are eighteen internal PCs, four internal servers, and one Internet-User.
- [ ] All VLANs 10, 11, 12, 20, 21, 22, 30, 31, 32, 99, 100, and 999 have been created.
- [ ] User ports are Access and in the correct VLAN.
- [ ] Three Trunks are active.
- [ ] The Native VLAN on both sides of all Trunks is 999.

## Layer 3 and Services

- [ ] All SVIs have the exact Gateway from the table.
- [ ] The CORE-to-EDGE link is `10.10.0.0/30`.
- [ ] OSPF Process 1 and Area 0 are enabled on both devices.
- [ ] The OSPF adjacency is in the FULL state.
- [ ] The Default Route from R-EDGE has been advertised in OSPF.
- [ ] There is one DHCP Pool for each user VLAN.
- [ ] Lease is 24 hours and DNS is `10.10.100.13`.
- [ ] Network Team members and servers are Static.
- [ ] Five internal DNS records have been created.
- [ ] TEST has no record in the public DNS.

## NAT and Security

- [ ] PAT translates all user VLANs to `203.0.113.10`.
- [ ] `203.0.113.20:80` goes to `WEB1:80`.
- [ ] `203.0.113.20:8080` goes to `WEB2:80`.
- [ ] There is no NAT for TEST.
- [ ] QA has access to TEST through HTTP.
- [ ] Non-QA has no access to TEST.
- [ ] On the internal network, Finance has only permitted HTTP and DNS toward the Server Farm, has no access to TEST, and on the internet, only its HTTP and DNS pass through PAT.
- [ ] Only the Network Team is allowed SSH/Telnet management of the equipment.
- [ ] The external ACL allows only public web and the necessary PAT responses through.

## Tests and Submission

- [ ] `show vlan brief` has been captured.
- [ ] `show interfaces trunk` has been captured.
- [ ] `show ip interface brief` has been captured.
- [ ] `ipconfig` for two VLANs has been captured.
- [ ] `show ip route` and `show ip ospf neighbor` have been captured.
- [ ] Ping from VLAN 10 to VLAN 22 is successful.
- [ ] Ping from VLAN 10 to WEB1 is successful.
- [ ] Both nslookup tests are successful.
- [ ] Both internal pages open by domain name.
- [ ] The PAT entry has been displayed.
- [ ] WEB1 and WEB2 open from Internet-User.
- [ ] QA is allowed to access TEST and non-QA is blocked.
- [ ] Ordinary-user SSH is denied and Network Team SSH is established.
- [ ] TEST is not accessible from the internet.
- [ ] The ACL counters have been displayed.
- [ ] The final `.pkt` file has been saved.
- [ ] The video is at most 10 minutes and shows the tests in order.

## List of 32 Checkpoints

- [ ] `F-01-topology.png`
- [ ] `F-02-edge-isp-links.png`
- [ ] `F-03-core-vlans.png`
- [ ] `F-04-core-trunks.png`
- [ ] `F-05-core-svis.png`
- [ ] `F-06-dhcp-bindings.png`
- [ ] `F-07-two-client-ipconfig.png`
- [ ] `F-08-server-addresses.png`
- [ ] `F-09-internal-dns-records.png`
- [ ] `F-10-external-dns-records.png`
- [ ] `F-11-ospf-neighbor.png`
- [ ] `F-12-core-routing-table.png`
- [ ] `F-13-pat-translation.png`
- [ ] `F-14-static-port-forwarding.png`
- [ ] `F-15-server-acl-config.png`
- [ ] `F-16-internet-acl-config.png`
- [ ] `F-17-management-vty-acl.png`
- [ ] `F-18-video-vlan-trunk-svi.png`
- [ ] `F-19-video-dhcp-two-vlans.png`
- [ ] `F-20-video-ospf-and-pings.png`
- [ ] `F-21-video-internal-dns.png`
- [ ] `F-22-video-internal-web-pages.png`
- [ ] `F-23-video-pat.png`
- [ ] `F-24-video-public-web1.png`
- [ ] `F-25-video-public-web2.png`
- [ ] `F-26-video-static-nat-table.png`
- [ ] `F-27-video-qa-test-allowed.png`
- [ ] `F-28-video-nonqa-test-blocked.png`
- [ ] `F-29-video-ordinary-ssh-denied.png`
- [ ] `F-30-video-network-ssh-allowed.png`
- [ ] `F-31-video-internet-test-blocked.png`
- [ ] `F-32-video-acl-counters.png`
