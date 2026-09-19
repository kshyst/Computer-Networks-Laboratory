# CNL8 — Experiment 1: Access Control Lists

**Group 6 · Class 2 · Cisco Packet Tracer · English standalone walkthrough**

This guide covers assignment experiment 1, steps **1–10**, in order. Build a new topology for this experiment; do not combine it with the DHCP Snooping experiment. The commands and outcome tables below are instructions and **expected results**, not a claim that Packet Tracer was run or that screenshots or `.pkt` files already exist. Record your actual results as you work.

## Before you begin: source corrections and conventions

- The source header says “seventh experiment,” but the user's assignment identity is **CNL8**. Use CNL8 and experiment number **8** in submission naming; experiment 1 here means the ACL part within CNL8.
- The source's references to Figures 2 and 6 in the ACL setup are inconsistent. Use the ACL topology described in step 1 below (the source's ACL overview, Figure 1), not the DHCP diagram.
- Add group number **6** to the third octet of every diagram address: original third octets `0`, `1`, and `2` become `6`, `7`, and `8`. Class number 2 does not change addresses. All networks use `/24`, mask `255.255.255.0`.
- The source labels R1's ports `F0/0` (transit), `F1/0` (LAN1), and `F0/1` (LAN2). Use **Cisco 2911** routers with three built-in routed GigabitEthernet ports and the explicit mapping below. Do not enter FastEthernet interface names on this model.
- Correct the source typo `aaccess-list` to `access-list`. Source step 6's bare “1” means PC1.
- The source counter example targets `SW1(config-if)`. ACLs are on **R1**; inspect them in R1 privileged EXEC mode with `show access-lists`.
- The source does not specify attachments for the extended ACLs. Apply them **inbound on both R1 LAN interfaces**, preserving standard ACL 10 **outbound on the transit interface**. A single interface/direction cannot run both IPv4 ACLs at once; putting 101 on the transit outbound direction would replace ACL 10 and break the required PC3 policy.
- The exact VTY password is **`NetLab`**, case-sensitive, as required by the assignment. This is an isolated lab-only credential; Telnet is plaintext and must not be used this way in production.

### How to use command blocks and evidence checkpoints

Router/switch blocks begin from the normal user EXEC CLI; `enable` enters privileged EXEC, and `configure terminal` enters global configuration. Each configuration block exits with `end` and saves with `write memory`. If already in configuration mode, enter `end` before starting another block. If the initial setup dialog appears on a new router, answer **no**, then press Enter to reach the CLI. Device prompts are intentionally omitted from executable blocks.

PC blocks run in **Desktop → Command Prompt** and do not use IOS configuration commands. A successful Telnet login is a temporary R2 session: capture it and type `exit` before continuing PC commands. Never paste subsequent PC tests into the remote router CLI.

Save screenshots at each inline **Checkpoint** before moving to the next stage. Basenames are fixed. For multi-window checkpoints, use the explicitly assigned `-a`, `-b`, `-c` frames; retain additional `-d`, `-e`, etc. if scrolling requires them. These frames belong to one base checkpoint, not extra checklist items. Keep device names and tested addresses readable. Failed ping text can be a timeout or destination-unreachable response; describe the actual output rather than inventing a specific error.

## Step 1 — Build, address, and route the topology

### 1.1 Place devices and cable them

Place two **2911** routers named **R1** and **R2**, two **2960-24TT** switches named **SW1** and **SW2**, and three **PC-PT** endpoints named **PC1**, **PC2**, **PC3**. Use copper straight-through cables for PC–switch and switch–router links. Use copper crossover for the router–router Ethernet link (Packet Tracer's automatic cable selection is also acceptable).

| Endpoint A | Endpoint B | Role |
|---|---|---|
| PC1 FastEthernet0 | SW1 FastEthernet0/1 | LAN1 host |
| PC2 FastEthernet0 | SW1 FastEthernet0/2 | LAN1 host |
| SW1 GigabitEthernet0/1 | R1 GigabitEthernet0/1 | LAN1 gateway |
| PC3 FastEthernet0 | SW2 FastEthernet0/1 | LAN2 host |
| SW2 GigabitEthernet0/1 | R1 GigabitEthernet0/2 | LAN2 gateway |
| R1 GigabitEthernet0/0 | R2 GigabitEthernet0/0 | Transit |

| Diagram role | Source port on R1 | Port used on 2911 R1 |
|---|---|---|
| Transit toward R2 | F0/0 | GigabitEthernet0/0 |
| LAN1 toward SW1 | F1/0 | GigabitEthernet0/1 |
| LAN2 toward SW2 | F0/1 | GigabitEthernet0/2 |

R2's diagram F0/0 becomes R2 GigabitEthernet0/0. Its other routed ports remain unused. No switch management IP, trunk, VLAN change, NAT, or routing protocol is needed: each switch is a separate default-VLAN Layer-2 LAN.

### 1.2 Addressing reference and actual PC configuration

**Reference table, not an automatic configuration action:** configure the PCs manually below; router addresses are entered in section 1.3.

| Device/interface | Original diagram address | Group 6 address | Mask | Default gateway |
|---|---|---|---|---|
| PC1 FastEthernet0 | 10.0.1.10 | 10.0.7.10 | 255.255.255.0 | 10.0.7.1 |
| PC2 FastEthernet0 | 10.0.1.11 | 10.0.7.11 | 255.255.255.0 | 10.0.7.1 |
| PC3 FastEthernet0 | 10.0.2.10 | 10.0.8.10 | 255.255.255.0 | 10.0.8.1 |
| R1 G0/1, LAN1 | 10.0.1.1 | 10.0.7.1 | 255.255.255.0 | Not applicable |
| R1 G0/2, LAN2 | 10.0.2.1 | 10.0.8.1 | 255.255.255.0 | Not applicable |
| R1 G0/0, transit | 10.0.0.1 | 10.0.6.1 | 255.255.255.0 | Not applicable |
| R2 G0/0, transit | 10.0.0.2 | 10.0.6.2 | 255.255.255.0 | Not applicable |

On **each PC**, open **Desktop → IP Configuration**, select **Static**, and enter that PC's group 6 IPv4 address, subnet mask, and default gateway exactly as listed. DNS is unnecessary for numeric-IP tests; leave it unset.

**Checkpoint `e1-01-pc-addressing.png`:** immediately capture PC1's IP Configuration as `e1-01-pc-addressing-a.png`, PC2's as `-b.png`, and PC3's as `-c.png`. Show device identity, address, mask, and gateway in each frame.

### 1.3 Configure router interfaces and return routes

**R1 → CLI, user EXEC to global configuration:**

```text
enable
configure terminal
hostname R1
interface GigabitEthernet0/0
 description Transit-to-R2
 ip address 10.0.6.1 255.255.255.0
 no shutdown
 exit
interface GigabitEthernet0/1
 description LAN1-to-SW1
 ip address 10.0.7.1 255.255.255.0
 no shutdown
 exit
interface GigabitEthernet0/2
 description LAN2-to-SW2
 ip address 10.0.8.1 255.255.255.0
 no shutdown
 exit
end
write memory
show ip interface brief
show ip route
```

**Checkpoint `e1-02-r1-baseline.png`:** capture R1's interface summary and route table now; use `-a` for interface state and `-b` for routes if needed. The three used ports should be `up/up`, with connected networks `10.0.6.0/24`, `10.0.7.0/24`, and `10.0.8.0/24`.

**R2 → CLI, user EXEC to global configuration:**

```text
enable
configure terminal
hostname R2
interface GigabitEthernet0/0
 description Transit-to-R1
 ip address 10.0.6.2 255.255.255.0
 no shutdown
 exit
ip route 10.0.7.0 255.255.255.0 10.0.6.1
ip route 10.0.8.0 255.255.255.0 10.0.6.1
end
write memory
show ip interface brief
show ip route
```

**Checkpoint `e1-03-r2-baseline.png`:** capture R2's `up/up` G0/0 address and both static routes via `10.0.6.1`; split interface summary (`-a`) and routes (`-b`) if needed. These return routes are essential: R2 must know how to reply to both PC subnets.

R1 needs no static route because all three subnets are directly connected. Switches require no CLI changes for this topology. Wait for link indicators to turn green, add readable device/port/subnet labels to the Logical workspace, and arrange the diagram to show PC1/PC2 → SW1 → R1, PC3 → SW2 → R1, and R1 → R2.

**Checkpoint `e1-04-topology.png`:** capture the complete labeled topology after links are green. Include all seven devices and the transformed addresses.

**Step 1 evidence:** `e1-01-pc-addressing.png`, `e1-02-r1-baseline.png`, `e1-03-r2-baseline.png`, `e1-04-topology.png`.

## Step 2 — Prove unrestricted baseline connectivity

No ACL is attached yet. Run the following full baseline, not only a single PC-to-router ping. Each PC tests both peers, every active R1 interface, and R2. Repeat a first ping if ARP resolution causes initial loss; persistent loss must be fixed before applying ACLs.

**PC1 → Desktop → Command Prompt:**

```text
ping 10.0.7.11
ping 10.0.8.10
ping 10.0.7.1
ping 10.0.8.1
ping 10.0.6.1
ping 10.0.6.2
```

**Checkpoint `e1-05-baseline-pc1.png`:** capture PC1's six destinations and actual replies immediately; use `-a` for peers, `-b` for R1 LAN addresses, and `-c` for R1 transit/R2 if necessary.

**PC2 → Desktop → Command Prompt:**

```text
ping 10.0.7.10
ping 10.0.8.10
ping 10.0.7.1
ping 10.0.8.1
ping 10.0.6.1
ping 10.0.6.2
```

**Checkpoint `e1-06-baseline-pc2.png`:** capture the same baseline categories for PC2, with `-a` peers, `-b` R1 LAN addresses, and `-c` R1 transit/R2.

**PC3 → Desktop → Command Prompt:**

```text
ping 10.0.7.10
ping 10.0.7.11
ping 10.0.7.1
ping 10.0.8.1
ping 10.0.6.1
ping 10.0.6.2
```

**Checkpoint `e1-07-baseline-pc3.png`:** capture the same baseline categories for PC3, with `-a` peers, `-b` R1 LAN addresses, and `-c` R1 transit/R2.

| Baseline test | Expected result |
|---|---|
| PC1 → PC2 and PC3 | Both pass |
| PC2 → PC1 and PC3 | Both pass |
| PC3 → PC1 and PC2 | Both pass |
| Each PC → all three R1 interface addresses | All pass |
| Each PC → R2 10.0.6.2 | All pass |

Use **File → Save As** to save **`CNL8-G6-C2-E1-01-baseline.pkt`**. Router `write memory` saves IOS startup configuration; Packet Tracer **Save As** saves the complete simulator project. Do both as instructed.

**Step 2 evidence:** `e1-05-baseline-pc1.png`, `e1-06-baseline-pc2.png`, `e1-07-baseline-pc3.png`.

## Step 3 — Apply standard numbered ACL 10

The original blocked subnet `10.0.2.0/24` becomes **`10.0.8.0/24`**. A standard ACL matches the source address only. Place it on R1's R2-facing G0/0 **outbound**: LAN2 packets destined across the transit link are blocked, while LAN2-to-LAN1 traffic never exits that port. `0.0.0.255` is the wildcard for a `/24`; `permit any` prevents the implicit final deny from blocking all other sources.

**R1 → CLI, user EXEC to global configuration:**

```text
enable
configure terminal
access-list 10 deny 10.0.8.0 0.0.0.255
access-list 10 permit any
interface GigabitEthernet0/0
 ip access-group 10 out
 exit
end
write memory
show access-lists
show ip interface GigabitEthernet0/0
```

**Checkpoint `e1-08-standard-attachment.png`:** capture ACL 10's two rules and G0/0's **Outgoing access list is 10**. Use `-a` for ACL definition and `-b` for interface attachment if needed. Definition alone is not proof of enforcement.

**Step 3 evidence:** `e1-08-standard-attachment.png`.

## Step 4 — Test standard ACL behavior

**PC1 → Desktop → Command Prompt:**

```text
ping 10.0.6.2
ping 10.0.7.11
ping 10.0.8.10
```

**Checkpoint `e1-09-standard-pc1.png`:** capture R2 and both peer pings; all are expected to pass. Split `-a` R2 and `-b` peers if necessary.

**PC2 → Desktop → Command Prompt:**

```text
ping 10.0.6.2
ping 10.0.7.10
ping 10.0.8.10
```

**Checkpoint `e1-10-standard-pc2.png`:** capture R2 and both peer pings; all are expected to pass. Split `-a` R2 and `-b` peers if necessary.

**PC3 → Desktop → Command Prompt:**

```text
ping 10.0.6.2
ping 10.0.7.10
ping 10.0.7.11
```

**Checkpoint `e1-11-standard-pc3.png`:** capture the expected failed R2 ping and successful PC1/PC2 pings. Split `-a` R2 and `-b` peers if necessary.

| Standard stage | PC1 | PC2 | PC3 |
|---|---|---|---|
| Ping R2 10.0.6.2 | Pass | Pass | Blocked by ACL 10 |
| Ping each of the other PCs | Pass | Pass | Pass |

Save As **`CNL8-G6-C2-E1-02-standard.pkt`**. Leave ACL 10 and its attachment unchanged for every later step.

**Step 4 evidence:** `e1-09-standard-pc1.png`, `e1-10-standard-pc2.png`, `e1-11-standard-pc3.png`.

## Step 5 — Enable R2 Telnet and add extended numbered ACL 101

### 5.1 Enable the actual destination service

**R2 → CLI, user EXEC to global configuration:**

```text
enable
configure terminal
line vty 0 4
 password NetLab
 login
 transport input telnet
 exit
end
write memory
show running-config
```

**Checkpoint `e1-12-r2-telnet-service.png`:** immediately capture R2's VTY configuration showing `login`, Telnet transport, and the lab password setting (it may be encoded if password encryption was already enabled). Scroll to `line vty 0 4`; no production credentials should be present. The five configured lines are sufficient for sequential tests; close each successful session.

### 5.2 Filter new PC traffic on both LAN ingress ports

ACL 101 permits PC1 TCP traffic to R2 destination port 23, denies other sources to that same service, then permits other IP traffic. Both LAN interfaces need it so PC3's attempted Telnet is subject to the same service policy. ACL 10 remains a separate later check on traffic exiting G0/0.

**R1 → CLI, user EXEC to global configuration:**

```text
enable
configure terminal
access-list 101 permit tcp host 10.0.7.10 host 10.0.6.2 eq telnet
access-list 101 deny tcp any host 10.0.6.2 eq telnet
access-list 101 permit ip any any
interface GigabitEthernet0/1
 ip access-group 101 in
 exit
interface GigabitEthernet0/2
 ip access-group 101 in
 exit
end
write memory
show access-lists
show ip interface GigabitEthernet0/1
show ip interface GigabitEthernet0/2
show ip interface GigabitEthernet0/0
```

**Checkpoint `e1-13-numbered-attachments.png`:** capture ACLs 10 and 101 (`-a`), G0/1 inbound 101 (`-b`), G0/2 inbound 101 (`-c`), and G0/0 outbound 10 (`-d`). Do not proceed if the standard attachment has disappeared.

**Step 5 evidence:** `e1-12-r2-telnet-service.png`, `e1-13-numbered-attachments.png`.

## Step 6 — Test the combined numbered ACLs

First run the pings on each PC, then attempt Telnet. On PC1, when the password prompt appears, type **NetLab** and press Enter; password input is not echoed. A password prompt alone proves reachability, not successful authentication. A visible R2 user EXEC prompt after login proves an application session. Capture before entering `exit`. On PC2 and PC3, allow the connection attempt to finish/timed out; no login should be possible.

**PC1 → Desktop → Command Prompt:**

```text
ping 10.0.6.2
ping 10.0.7.11
ping 10.0.8.10
telnet 10.0.6.2
```

**Checkpoint `e1-14-numbered-pc1.png`:** capture successful R2 ping (`-a`), both peer pings (`-b`), and authenticated Telnet session (`-c`). Then type `exit` in that session to return to the PC command prompt.

**PC2 → Desktop → Command Prompt:**

```text
ping 10.0.6.2
ping 10.0.7.10
ping 10.0.8.10
telnet 10.0.6.2
```

**Checkpoint `e1-15-numbered-pc2.png`:** capture successful R2 ping (`-a`), successful peer pings (`-b`), and failed Telnet attempt (`-c`).

**PC3 → Desktop → Command Prompt:**

```text
ping 10.0.6.2
ping 10.0.7.10
ping 10.0.7.11
telnet 10.0.6.2
```

**Checkpoint `e1-16-numbered-pc3.png`:** capture failed R2 ping (`-a`), successful peer pings (`-b`), and failed Telnet attempt (`-c`).

| Numbered extended stage | PC1 | PC2 | PC3 |
|---|---|---|---|
| Ping R2 10.0.6.2 | Pass | Pass | Blocked by ACL 10 |
| Telnet R2 10.0.6.2 | Allowed; authenticate with NetLab | Blocked by ACL 101 | Blocked by ACL 101 |
| Ping each other PC | Pass | Pass | Pass |

ACL 101's final IP permit allows PC3's echo request through LAN ingress, but ACL 10 then denies it at transit egress. PC1/PC2 inter-LAN pings are not destined for R2 and remain allowed. PC1↔PC2 same-LAN traffic is switched locally, not routed through these ACLs.

**Step 6 evidence:** `e1-14-numbered-pc1.png`, `e1-15-numbered-pc2.png`, `e1-16-numbered-pc3.png`.

## Step 7 — Answer the permitted Telnet packet-count question

**R1 → CLI, user EXEC to privileged EXEC:**

```text
enable
show access-lists
```

**Checkpoint `e1-17-telnet-permit-counter.png`:** capture extended ACL 101 and the match count on the exact rule `permit tcp host 10.0.7.10 host 10.0.6.2 eq telnet` immediately after the step 6 sessions. Make the rule and its displayed counter readable; include deny and final-permit lines for context.

Answer using **your actual displayed count**: “At the recorded checkpoint, ACL 101's PC1-to-R2 Telnet permit entry showed ___ matches.” Fill the blank only from the output. Do not use the count on ACL 10's `permit any` or ACL 101's `permit ip any any`, and do not count successful login sessions instead of packets.

There is no predetermined correct numeric value: TCP handshakes, acknowledgments, interactive keystrokes, retransmissions, teardown, and previous tests can all change the number. It represents matching packets in the filtered PC-to-R2 direction, not both directions of all Telnet conversations. The same ACL on both LAN interfaces reports matches for that ACL, rather than a separate count per attachment. For an isolated test, record the same entry before and after one new PC1 session and report the measured difference as well as the cumulative value; do not claim that the old counter was reset.

If your Packet Tracer version omits counters, state “match counter not exposed in this version,” capture that actual output, and use Simulation mode's TCP event/PDU views to document permitted PC1→R2 port-23 traffic. A Simulation event count is separate evidence, not a fabricated IOS ACL counter.

Save As **`CNL8-G6-C2-E1-03-numbered.pkt`**. Keep the screenshot: counters may not survive reopening a project even when configuration does.

**Step 7 evidence:** `e1-17-telnet-permit-counter.png`.

## Step 8 — Detach ACL 101 without deleting its definition

Detach from **both** LAN interfaces. Do not run `no access-list 101`, and do not detach ACL 10.

**R1 → CLI, user EXEC to global configuration:**

```text
enable
configure terminal
interface GigabitEthernet0/1
 no ip access-group 101 in
 exit
interface GigabitEthernet0/2
 no ip access-group 101 in
 exit
end
write memory
show access-lists
show ip interface GigabitEthernet0/1
show ip interface GigabitEthernet0/2
show ip interface GigabitEthernet0/0
```

**Checkpoint `e1-18-numbered-detached.png`:** capture ACL 101 still defined with its three entries (`-a`), both LAN interfaces with no incoming ACL (`-b` G0/1, `-c` G0/2), and G0/0 still outbound ACL 10 (`-d`). A defined but unattached ACL does not filter traffic.

| Temporary detached stage | PC1 | PC2 | PC3 |
|---|---|---|---|
| Ping R2 | Pass | Pass | Blocked by ACL 10 |
| Telnet R2 | Allowed | Allowed | Blocked by ACL 10 |
| Ping both peers | Pass | Pass | Pass |

This is an intentional intermediate configuration, not the final security policy. Save As **`CNL8-G6-C2-E1-04-numbered-detached.pkt`** before adding the named ACL. If you make optional Telnet checks here, close all sessions before step 9; already established sessions are not a substitute for fresh policy tests.

**Step 8 evidence:** `e1-18-numbered-detached.png`.

## Step 9 — Configure and attach named extended ACL NetLab

Create rules in this exact order. Permit PC1 Telnet before the general Telnet deny, and permit PC2 ICMP echo before the general echo deny. The final IP permit preserves other IP traffic subject to the still-active standard ACL. The ICMP rule denies **echo requests to R2**, not every ICMP type or every ICMP destination.

**R1 → CLI, user EXEC to global configuration:**

```text
enable
configure terminal
ip access-list extended NetLab
 permit tcp host 10.0.7.10 host 10.0.6.2 eq telnet
 deny tcp any host 10.0.6.2 eq telnet
 permit icmp host 10.0.7.11 host 10.0.6.2 echo
 deny icmp any host 10.0.6.2 echo
 permit ip any any
 exit
interface GigabitEthernet0/1
 ip access-group NetLab in
 exit
interface GigabitEthernet0/2
 ip access-group NetLab in
 exit
end
write memory
show access-lists
show ip interface GigabitEthernet0/1
show ip interface GigabitEthernet0/2
show ip interface GigabitEthernet0/0
```

**Checkpoint `e1-19-named-attachments.png`:** capture NetLab's five ordered entries, ACL 10, and the retained ACL 101 definition (`-a`, additional frames if needed); show G0/1 inbound NetLab (`-b`), G0/2 inbound NetLab (`-c`), and G0/0 outbound 10 (`-d`). ACL 101 must no longer be attached to either LAN interface.

| R1 interface | Incoming ACL at this stage | Outgoing ACL at this stage |
|---|---|---|
| G0/0, transit | None | 10 |
| G0/1, LAN1 | NetLab | None |
| G0/2, LAN2 | NetLab | None |

**Step 9 evidence:** `e1-19-named-attachments.png`.

## Step 10 — Verify named ACL results and save the final stage

Open fresh Telnet sessions. PC1 must still log in even though its R2 ping is now blocked: ping and Telnet are different protocols and are intentionally treated differently.

**PC1 → Desktop → Command Prompt:**

```text
ping 10.0.6.2
ping 10.0.7.11
ping 10.0.8.10
telnet 10.0.6.2
```

**Checkpoint `e1-20-named-pc1.png`:** capture blocked R2 ping (`-a`), successful peer pings (`-b`), and successful fresh Telnet login with NetLab (`-c`). After capture, type `exit` in the remote R2 session.

**PC2 → Desktop → Command Prompt:**

```text
ping 10.0.6.2
ping 10.0.7.10
ping 10.0.8.10
telnet 10.0.6.2
```

**Checkpoint `e1-21-named-pc2.png`:** capture successful R2 ping (`-a`), successful peer pings (`-b`), and blocked fresh Telnet attempt (`-c`).

**PC3 → Desktop → Command Prompt:**

```text
ping 10.0.6.2
ping 10.0.7.10
ping 10.0.7.11
telnet 10.0.6.2
```

**Checkpoint `e1-22-named-pc3.png`:** capture blocked R2 ping (`-a`), successful peer pings (`-b`), and blocked Telnet attempt (`-c`).

| Final named stage | PC1 | PC2 | PC3 |
|---|---|---|---|
| Ping R2 10.0.6.2 | Blocked by NetLab echo deny | Allowed by NetLab echo permit | Blocked by NetLab echo deny |
| Telnet R2 10.0.6.2 | Allowed by NetLab TCP permit | Blocked by NetLab TCP deny | Blocked by NetLab TCP deny |
| Ping each of the other PCs | Pass | Pass | Pass |

Although NetLab now drops PC3's R2 ping first, ACL 10 must still be present and attached. It continues to block other LAN2-originated IP traffic toward R2 that NetLab's final permit would otherwise allow. “Preserve other communications” does not undo the earlier standard-ACL restriction.

**R1 → CLI, user EXEC to privileged EXEC, final audit/save:**

```text
enable
show access-lists
show ip interface GigabitEthernet0/0
show ip interface GigabitEthernet0/1
show ip interface GigabitEthernet0/2
show running-config
write memory
```

**Checkpoint `e1-23-final-r1-audit.png`:** immediately capture actual rule counters/definitions (`-a`), G0/0 outbound 10 (`-b`), G0/1 inbound NetLab (`-c`), and G0/2 inbound NetLab (`-d`). Preserve readable running-config excerpts as additional frames if needed to show ACL 101 retained but detached. Do not infer failure from a zero final-stage ACL 10 deny counter: NetLab can reject a tested packet before it reaches ACL 10.

**R2 → CLI, user EXEC to privileged EXEC, final audit/save:**

```text
enable
show ip route
show running-config
write memory
```

**Checkpoint `e1-24-final-r2-audit.png`:** capture both return routes (`-a`) and the VTY Telnet/login configuration (`-b`).

Save As **`CNL8-G6-C2-E1-05-named-final.pkt`**. Confirm the final filename in Packet Tracer. Reopen this saved final project, inspect the attachments again, and repeat the step 10 tests; record any discrepancy rather than marking unchecked tests as passed. Keep all earlier stage projects distinct instead of overwriting them.

**Step 10 evidence:** `e1-20-named-pc1.png`, `e1-21-named-pc2.png`, `e1-22-named-pc3.png`, `e1-23-final-r1-audit.png`, `e1-24-final-r2-audit.png`.

## Troubleshooting without changing the required policy

| Symptom | Check first |
|---|---|
| Same-LAN PC1↔PC2 fails at baseline | PC masks/IPs, switch cables/ports, green link state; R1 is not in this path |
| PCs reach own gateway but not the other LAN | R1 G0/1/G0/2 addresses and `no shutdown`; both PC default gateways |
| Baseline R2 ping fails while R1 is reachable | Transit link/addressing, R2's two static return routes, absence of premature ACL attachments |
| First ping loses a packet | Allow ARP/link convergence and repeat; do not interpret a single initial loss as proof of ACL denial |
| PC3 reaches R2 after standard ACL | ACL 10 source must be 10.0.8.0 with wildcard 0.0.0.255; attachment must be R1 G0/0 **out** |
| Inter-LAN pings fail after an extended ACL | Final `permit ip any any`, exact R2 host destination, correct rule order, and ingress ports |
| PC1 ping passes but Telnet fails in numbered stage | R2 VTY service/login/password, available VTY lines, source/destination of ACL 101's first entry |
| PC1 ping fails but Telnet passes in named stage | This is the required final outcome, not a defect |
| Telnet shows a password prompt but rejects login | Network connection reached R2; check case-sensitive NetLab authentication rather than blaming a packet ACL |
| ACL exists but traffic is not filtered | Inspect `show ip interface` for the attachment and direction; a definition alone is inactive |
| Command is rejected by your simulator | Check selected model, current CLI mode, exact spelling/port names, and record PT version/error; do not claim successful enforcement |

ACLs are first-match, top-to-bottom, with an implicit deny after the final entry. Never “fix” connectivity by deleting ACL 10 or replacing required denies with broad early permits. Use the saved baseline to distinguish addressing problems from policy problems.

## Submission notes

This walkthrough is not a completed report. In your report, explain standard versus extended ACL matching, attachment direction, coexistence of ACL 10 with each extended policy, the difference between detaching and deleting ACL 101, and the **measured** step 7 permit count or the documented counter-display limitation. Report actual pass/fail observations against the tables, including unexpected outcomes. Include every captured non-duplicate evidence frame used to support your answers; do not manufacture screenshots or observations.

The source requires references for outside material used to answer questions. Cite the provided assignment and identify any additional documentation used. The source's submission instruction is a single ZIP containing **all experiment-related files**, not just this ACL guide. Coordinate with the separate experiment 2 walkthrough/results when preparing the complete CNL8 submission.

Use the source's filename pattern `FullName-GroupNum-ClassNum-ExperimentNum.zip`, instantiated as **`FullName-6-2-8.zip`**; replace `FullName` with your actual required name spelling. Do not invent a student's name. Include the requested report, final experiment projects, stage projects/evidence needed to substantiate the progression, and other required experiment files. Do not submit CNL7 merely because the source header is mislabeled. This document does not create or claim to have created those student-run artifacts.

## Final project checklist

- [ ] `CNL8-G6-C2-E1-01-baseline.pkt` — all baseline tests complete, no ACL attached.
- [ ] `CNL8-G6-C2-E1-02-standard.pkt` — ACL 10 outbound on R1 G0/0.
- [ ] `CNL8-G6-C2-E1-03-numbered.pkt` — ACL 101 inbound on both LANs, ACL 10 retained; step 7 counter captured.
- [ ] `CNL8-G6-C2-E1-04-numbered-detached.pkt` — ACL 101 defined but detached from both LANs; ACL 10 retained.
- [ ] `CNL8-G6-C2-E1-05-named-final.pkt` — NetLab inbound on both LANs; ACL 10 outbound transit; ACL 101 retained but detached; reopened and retested.

## Complete screenshot checklist

There are **24 base screenshot checkpoints**. Any `-a`, `-b`, `-c`, `-d`, or further readability frames belong to their listed base and do not increase this base count. The base names here exactly match the inline checkpoints and step evidence lists.

### Setup and baseline — steps 1–2

- [ ] `e1-01-pc-addressing.png`
- [ ] `e1-02-r1-baseline.png`
- [ ] `e1-03-r2-baseline.png`
- [ ] `e1-04-topology.png`
- [ ] `e1-05-baseline-pc1.png`
- [ ] `e1-06-baseline-pc2.png`
- [ ] `e1-07-baseline-pc3.png`

### Standard ACL — steps 3–4

- [ ] `e1-08-standard-attachment.png`
- [ ] `e1-09-standard-pc1.png`
- [ ] `e1-10-standard-pc2.png`
- [ ] `e1-11-standard-pc3.png`

### Numbered extended ACL, counter, and detachment — steps 5–8

- [ ] `e1-12-r2-telnet-service.png`
- [ ] `e1-13-numbered-attachments.png`
- [ ] `e1-14-numbered-pc1.png`
- [ ] `e1-15-numbered-pc2.png`
- [ ] `e1-16-numbered-pc3.png`
- [ ] `e1-17-telnet-permit-counter.png`
- [ ] `e1-18-numbered-detached.png`

### Named extended ACL and final audit — steps 9–10

- [ ] `e1-19-named-attachments.png`
- [ ] `e1-20-named-pc1.png`
- [ ] `e1-21-named-pc2.png`
- [ ] `e1-22-named-pc3.png`
- [ ] `e1-23-final-r1-audit.png`
- [ ] `e1-24-final-r2-audit.png`
