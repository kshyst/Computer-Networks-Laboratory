# CNL5 Experiment 1 — complete Packet Tracer walkthrough

## Exact scope

This walkthrough covers **only Experiment 1** of `آزمایش پنجم (3).pdf`. Do not
build Experiment 2 and do not configure multiple OSPF areas.

It covers every Experiment 1 requirement:

1. Reuse and verify the Experiment 4 topology and group-based addressing.
2. Make three independent Packet Tracer files: OSPF, EIGRP, and RIPv2.
3. Inspect and explain routing-control packets in Simulation mode.
4. Measure and compare `ping` delay under identical conditions.
5. originate a default route toward the Internet router.
6. keep every dynamic routing protocol off the external Internet link and keep
   internal routes out of the Internet router.
7. investigate 50/50 load sharing through R2 and R5, including the real-router
   method that Packet Tracer cannot faithfully demonstrate.
8. make R4 prefer R3 for all non-connected traffic while retaining R5 as a
   backup.
9. turn off R2, measure convergence, compare the protocols, and explain the
   observed result.

The professor's extra instructions are treated as mandatory: manipulate link
`bandwidth` and `delay`, prove metric-sensitive rerouting rather than giving
only theory, and generate two simultaneous 1000-packet streams.

This file is an execution guide and evidence checklist. It is **not** the final
report. Do not create `report.txt` or `report.tex` yet.

## What already exists on this computer

- Packet Tracer 7.0 is installed.
- The correct starting topology is `../CNL4/3.pkt`, not `../CNL4/1.pkt`.
- The prior CNL4 work identifies the group number as **6**.
- R1, R4, and R5 already have `NM-2FE2W` modules and the required third
  FastEthernet interface.
- The old CNL4 file contains static routes. Those routes must be removed before
  any dynamic protocol is enabled, or the static routes will win because their
  administrative distance is lower.
- The final CNL4 file changed the external addresses to `213.80.17.5/24` and
  `213.80.17.6/24`. CNL5 Figure 1 explicitly uses `213.80.11.4/24` and
  `213.80.11.5/24`, so restore the Figure 1 values. The group-number addition
  applies only to the `10.10.x.y` addresses in this experiment.

## Files to finish with

The three files to submit for Experiment 1 are:

- `KiarashShojaei-6-2-5-OSPF.pkt`
- `KiarashShojaei-6-2-5-EIGRP.pkt`
- `KiarashShojaei-6-2-5-RIP.pkt`

You may keep `KiarashShojaei-6-2-5-BASE.pkt` as a private safety copy, but do
not put it in the final submission unless the professor asks for it.

Create these evidence directories inside `CNL5` before starting:

```text
screenshots/
├── common/
├── ospf/
├── eigrp/
└── rip/
```

## Screenshot rules

For every named checkpoint below:

1. Use the exact filename.
2. Keep the device name or CLI prompt visible.
3. Keep the complete command and the relevant output visible.
4. For route-table evidence, do not crop out the route code, metric, next hop,
   or outgoing interface.
5. For Simulation evidence, show the Event List row and the expanded packet
   fields together whenever possible.
6. Never report an expected value as measured. Record the actual Packet Tracer
   output.
7. If one terminal window cannot show all relevant lines legibly, take two
   screenshots and add `-a` and `-b` before `.png`.

On Ubuntu, `Alt+Print Screen` captures the active window. Use a normal desktop
screenshot when two Packet Tracer device windows must be visible together.

## Addressing and interface map

The CNL5 rule is `z = x + group_number`. With group number 6, the internal
subnets are as follows.

| Device | Interface | Connected to | IPv4 address | Mask |
|---|---|---|---|---|
| PC1 | FastEthernet0 | SW1/R1 LAN | `10.10.6.1` | `255.255.255.0` |
| R1 | Fa0/0 | PC1 LAN | `10.10.6.2` | `255.255.255.0` |
| R1 | Fa0/1 | R2 | `10.10.8.1` | `255.255.255.0` |
| R1 | Fa1/0 | R5 | `10.10.11.1` | `255.255.255.0` |
| R2 | Fa0/0 | R1 | `10.10.8.2` | `255.255.255.0` |
| R2 | Fa0/1 | R3 | `10.10.9.1` | `255.255.255.0` |
| R3 | Fa0/0 | R2 | `10.10.9.2` | `255.255.255.0` |
| R3 | Fa0/1 | R4 | `10.10.10.1` | `255.255.255.0` |
| R4 | Fa0/0 | PC3 LAN | `10.10.16.2` | `255.255.255.0` |
| R4 | Fa0/1 | R3 | `10.10.10.2` | `255.255.255.0` |
| R4 | Fa1/0 | R5 | `10.10.12.1` | `255.255.255.0` |
| R5 | Fa0/0 | R1 | `10.10.11.2` | `255.255.255.0` |
| R5 | Fa0/1 | R4 | `10.10.12.2` | `255.255.255.0` |
| R5 | Fa1/0 | SW4/Internet LAN | `213.80.11.4` | `255.255.255.0` |
| Internet | verify with `show ip interface brief` | SW4/R5 LAN | `213.80.11.5` | `255.255.255.0` |
| PC3 | FastEthernet0 | SW3/R4 LAN | `10.10.16.1` | `255.255.255.0` |

PC default gateways:

- PC1: `10.10.6.2`
- PC3: `10.10.16.2`

The internal graph is:

```text
PC1 -- R1 -- R2 -- R3 -- R4 -- PC3
        \                 /
         -------- R5 -----
                   |
                Internet
```

R1-to-R4 therefore has two internal paths:

- upper path: R1-R2-R3-R4, three router-to-router links;
- lower path: R1-R5-R4, two router-to-router links.

---

# Phase A — prepare one clean common base

Complete this phase once, before configuring any routing protocol.

## A1. Open the correct topology and save a working base

1. Start Packet Tracer.
2. Click `File` -> `Open`.
3. Open `/home/kshyst/Desktop/University/CNL/CNL4/3.pkt`.
4. Confirm that the topology contains PC1, PC3, R1-R5, the Internet router,
   three switches, and both the upper and lower paths.
5. Click `File` -> `Save As`.
6. Save it as
   `/home/kshyst/Desktop/University/CNL/CNL5/KiarashShojaei-6-2-5-BASE.pkt`.
7. Work only in the new CNL5 copy. Never overwrite the CNL4 source.

Screenshot checkpoint:

- `screenshots/common/00-base-topology-opened.png` — show the whole topology,
  every device name, and all green links.

## A2. Verify every interface before changing routes

On each router, open `CLI`, press Enter if needed, then run:

```text
enable
show ip interface brief
```

Compare every address and interface with the table above. All used interfaces
must be `up/up`. If an interface is `administratively down`, enable it:

```text
configure terminal
interface <interface-name>
no shutdown
end
```

On R5, restore the CNL5 external address:

```text
enable
configure terminal
interface FastEthernet1/0
 ip address 213.80.11.4 255.255.255.0
 no shutdown
end
```

On the Internet router, first use `show ip interface brief` to identify the
connected FastEthernet interface. In the existing file it should be Fa0/0. If
the output shows another interface, substitute that real interface below:

```text
enable
configure terminal
interface FastEthernet0/0
 ip address 213.80.11.5 255.255.255.0
 no shutdown
end
```

Configure the PCs through `Desktop` -> `IP Configuration` using the address,
mask, and gateway values above.

Screenshot checkpoints:

- `screenshots/common/01-r1-interface-summary.png`
- `screenshots/common/02-r2-interface-summary.png`
- `screenshots/common/03-r3-interface-summary.png`
- `screenshots/common/04-r4-interface-summary.png`
- `screenshots/common/05-r5-interface-summary.png`
- `screenshots/common/06-internet-interface-summary.png`
- `screenshots/common/07-pc1-ip-configuration.png`
- `screenshots/common/08-pc3-ip-configuration.png`

Each router screenshot must show the full `show ip interface brief` result.

## A3. Remove every old static route

Do this on R1, R2, R3, R4, R5, and Internet.

1. Run:

   ```text
   show running-config | include ip route
   ```

2. If Packet Tracer 7 rejects the pipe, run `show running-config` and scroll to
   the global `ip route` lines.
3. For every displayed line, enter configuration mode and repeat that exact
   line with `no` in front. Example:

   ```text
   configure terminal
   no ip route 10.10.16.0 255.255.255.0 10.10.8.2
   end
   ```

4. Repeat the show command. It must return no `ip route` lines on all six
   routers at this point.
5. Also inspect the configuration for an old `router rip`, `router ospf`, or
   `router eigrp` block. Remove a routing process only if it is actually present.

Why this matters: leaving even one old static internal route would prevent a
fair OSPF/EIGRP/RIP comparison.

Screenshot checkpoints:

- `screenshots/common/09-r1-static-routes-cleared.png`
- `screenshots/common/10-r2-static-routes-cleared.png`
- `screenshots/common/11-r3-static-routes-cleared.png`
- `screenshots/common/12-r4-static-routes-cleared.png`
- `screenshots/common/13-r5-static-routes-cleared.png`
- `screenshots/common/14-internet-static-routes-cleared.png`

The command and its empty result must both be visible.

## A4. Configure the Internet edge without leaking internal routes

R5 is the border router. The dynamic protocol will run only on R5's two
internal interfaces, never on R5 Fa1/0 and never on the Internet router.

Because the Internet router is not allowed to know any `10.10.*` route, do
**not** recreate the CNL4 summary route on Internet. Use NAT overload on R5 so
the Internet router can reply to the directly connected address
`213.80.11.4` instead.

On R5:

```text
enable
configure terminal
access-list 1 permit 10.10.0.0 0.0.255.255
interface FastEthernet0/0
 ip nat inside
exit
interface FastEthernet0/1
 ip nat inside
exit
interface FastEthernet1/0
 ip nat outside
exit
ip nat inside source list 1 interface FastEthernet1/0 overload
ip route 0.0.0.0 0.0.0.0 213.80.11.5
end
```

Do not configure any default or internal static route on the Internet router.
Verify:

On R5:

```text
show ip route 0.0.0.0
show ip nat statistics
```

On Internet:

```text
show ip route
show ip protocols
```

Expected Internet route-table state: only its connected/local
`213.80.11.0/24` entries; no `10.10.*` route and no routing protocol.

Screenshot checkpoints:

- `screenshots/common/15-r5-default-and-nat.png`
- `screenshots/common/16-internet-has-no-internal-routes.png`
- `screenshots/common/17-internet-has-no-routing-protocol.png`

Do not proceed if Internet shows an OSPF, EIGRP, RIP, static `10.10.*`, or
summary `10.10.0.0/16` route.

## A5. Apply one identical artificial metric profile

The professor requires lower bandwidth and higher delay on links between
routers. Use this same baseline on **both ends of all five internal
router-to-router links**:

```text
bandwidth 10000
delay 1000
```

IOS interprets `bandwidth` in kbit/s and `delay` in tens of microseconds, so
this advertises 10 Mbit/s and 10 ms to routing protocols. Do not apply these
commands to either PC LAN or to the R5-Internet link.

Configure the interfaces below:

### R1

```text
enable
configure terminal
interface FastEthernet0/1
 bandwidth 10000
 delay 1000
exit
interface FastEthernet1/0
 bandwidth 10000
 delay 1000
end
```

### R2

```text
enable
configure terminal
interface FastEthernet0/0
 bandwidth 10000
 delay 1000
exit
interface FastEthernet0/1
 bandwidth 10000
 delay 1000
end
```

### R3

```text
enable
configure terminal
interface FastEthernet0/0
 bandwidth 10000
 delay 1000
exit
interface FastEthernet0/1
 bandwidth 10000
 delay 1000
end
```

### R4

```text
enable
configure terminal
interface FastEthernet0/1
 bandwidth 10000
 delay 1000
exit
interface FastEthernet1/0
 bandwidth 10000
 delay 1000
end
```

### R5

```text
enable
configure terminal
interface FastEthernet0/0
 bandwidth 10000
 delay 1000
exit
interface FastEthernet0/1
 bandwidth 10000
 delay 1000
end
```

On each router, use `show interfaces <interface>` and verify that the first
lines show the intended `BW` and `DLY`. Also verify `up, line protocol is up`.

Screenshot checkpoints:

- `screenshots/common/18-r1-artificial-link-metrics.png`
- `screenshots/common/19-r2-artificial-link-metrics.png`
- `screenshots/common/20-r3-artificial-link-metrics.png`
- `screenshots/common/21-r4-artificial-link-metrics.png`
- `screenshots/common/22-r5-artificial-link-metrics.png`

One screenshot per router may show its running-config interface blocks instead
of two separate `show interfaces` screens, provided both interfaces and both
values are legible.

Important accuracy note: on physical Cisco IOS, `bandwidth` and `delay` are
routing/QoS metadata and do not physically throttle an Ethernet interface.
Follow the professor's Packet Tracer procedure, but if CLI ping still reports
`<1 ms`, do not invent a delay. Record that output and also use Simulation
event times to compare the paths.

## A6. Test only directly connected links

Run these pings from the router CLI:

- R1: `ping 10.10.8.2` and `ping 10.10.11.2`
- R2: `ping 10.10.8.1` and `ping 10.10.9.2`
- R3: `ping 10.10.9.1` and `ping 10.10.10.2`
- R4: `ping 10.10.10.1` and `ping 10.10.12.2`
- R5: `ping 10.10.11.1`, `ping 10.10.12.1`, and `ping 213.80.11.5`
- PC1: `ping 10.10.6.2`
- PC3: `ping 10.10.16.2`

All must succeed. PC1-to-PC3 is expected to fail before a dynamic protocol is
configured.

Screenshot checkpoints:

- `screenshots/common/23-router-adjacency-pings.png` — take `-a`, `-b`, etc. if
  necessary to keep all results readable.
- `screenshots/common/24-pc-gateway-pings.png`

## A7. Save the three independent projects

1. Save the clean base.
2. Use `File` -> `Save As` to create
   `KiarashShojaei-6-2-5-OSPF.pkt`.
3. Reopen the clean base and save as
   `KiarashShojaei-6-2-5-EIGRP.pkt`.
4. Reopen the clean base again and save as
   `KiarashShojaei-6-2-5-RIP.pkt`.
5. Verify in Files that all three are non-empty and have different names.

Screenshot checkpoint:

- `screenshots/common/25-three-protocol-files.png` — show the three filenames
  and sizes in the CNL5 folder.

---

# Phase B — configure OSPF

Open `KiarashShojaei-6-2-5-OSPF.pkt` and configure only OSPF process 1.

The `passive-interface default` pattern is deliberate: every interface starts
silent, and only internal router-to-router links are opened. This is stronger
proof against accidental Internet leakage than relying on memory.

## B1. OSPF configuration

### R1

```text
enable
configure terminal
router ospf 1
 router-id 1.1.1.1
 network 10.10.0.0 0.0.255.255 area 0
 passive-interface default
 no passive-interface FastEthernet0/1
 no passive-interface FastEthernet1/0
end
copy running-config startup-config
```

### R2

```text
enable
configure terminal
router ospf 1
 router-id 2.2.2.2
 network 10.10.0.0 0.0.255.255 area 0
 passive-interface default
 no passive-interface FastEthernet0/0
 no passive-interface FastEthernet0/1
end
copy running-config startup-config
```

### R3

```text
enable
configure terminal
router ospf 1
 router-id 3.3.3.3
 network 10.10.0.0 0.0.255.255 area 0
 passive-interface default
 no passive-interface FastEthernet0/0
 no passive-interface FastEthernet0/1
end
copy running-config startup-config
```

### R4

```text
enable
configure terminal
router ospf 1
 router-id 4.4.4.4
 network 10.10.0.0 0.0.255.255 area 0
 passive-interface default
 no passive-interface FastEthernet0/1
 no passive-interface FastEthernet1/0
end
copy running-config startup-config
```

### R5

```text
enable
configure terminal
router ospf 1
 router-id 5.5.5.5
 network 10.10.0.0 0.0.255.255 area 0
 passive-interface default
 no passive-interface FastEthernet0/0
 no passive-interface FastEthernet0/1
 default-information originate
end
copy running-config startup-config
```

Do not configure OSPF on Internet.

If this old Packet Tracer image rejects `passive-interface default`, remove
that line and explicitly make only these interfaces passive: R1 Fa0/0, R4
Fa0/0, and R5 Fa1/0. Keep the same internal interfaces active.

Screenshot checkpoints:

- `screenshots/ospf/01-r1-ospf-config.png`
- `screenshots/ospf/02-r2-ospf-config.png`
- `screenshots/ospf/03-r3-ospf-config.png`
- `screenshots/ospf/04-r4-ospf-config.png`
- `screenshots/ospf/05-r5-ospf-config.png`

Use `show running-config` and show the complete `router ospf 1` block in each.

## B2. Verify OSPF convergence and Internet isolation

Wait until all orange link indicators return to green, then run:

On R1 and R4:

```text
show ip ospf neighbor
show ip route ospf
show ip route 0.0.0.0
```

On R5:

```text
show ip ospf neighbor
show ip ospf interface brief
show ip route 0.0.0.0
```

R5 Fa1/0 must not appear as an OSPF interface. R1 and R4 should learn a default
route marked `O*E2`, ultimately originated by R5.

On Internet:

```text
show ip protocols
show ip route
```

It must still have no OSPF process and no `10.10.*` route.

From PC1, warm up ARP and verify both destinations:

```text
ping 10.10.16.1
ping -n 10 213.80.11.5
```

Immediately afterward, on R5 run:

```text
show ip nat translations
show ip nat statistics
```

Screenshot checkpoints:

- `screenshots/ospf/06-r1-neighbors-and-routes.png`
- `screenshots/ospf/07-r4-neighbors-and-routes.png`
- `screenshots/ospf/08-r5-external-interface-excluded.png`
- `screenshots/ospf/09-internet-still-isolated.png`
- `screenshots/ospf/10-pc1-internal-and-internet-ping.png`
- `screenshots/ospf/11-r5-nat-proof.png`

Do not continue until PC1-to-PC3 works, the default route exists internally,
and Internet remains unaware of the internal prefixes.

---

# Phase C — configure EIGRP

Open `KiarashShojaei-6-2-5-EIGRP.pkt`. Use classic EIGRP autonomous-system
number 100 on R1-R5 only.

## C1. EIGRP configuration

### R1

```text
enable
configure terminal
router eigrp 100
 no auto-summary
 network 10.10.0.0 0.0.255.255
 passive-interface default
 no passive-interface FastEthernet0/1
 no passive-interface FastEthernet1/0
end
copy running-config startup-config
```

### R2

```text
enable
configure terminal
router eigrp 100
 no auto-summary
 network 10.10.0.0 0.0.255.255
 passive-interface default
 no passive-interface FastEthernet0/0
 no passive-interface FastEthernet0/1
end
copy running-config startup-config
```

### R3

```text
enable
configure terminal
router eigrp 100
 no auto-summary
 network 10.10.0.0 0.0.255.255
 passive-interface default
 no passive-interface FastEthernet0/0
 no passive-interface FastEthernet0/1
end
copy running-config startup-config
```

### R4

```text
enable
configure terminal
router eigrp 100
 no auto-summary
 network 10.10.0.0 0.0.255.255
 passive-interface default
 no passive-interface FastEthernet0/1
 no passive-interface FastEthernet1/0
end
copy running-config startup-config
```

### R5

R5 has only one static route, the default route created in Phase A. Redistribute
that route with an explicit seed metric:

```text
enable
configure terminal
router eigrp 100
 no auto-summary
 network 10.10.0.0 0.0.255.255
 passive-interface default
 no passive-interface FastEthernet0/0
 no passive-interface FastEthernet0/1
 redistribute static metric 100000 1000 255 1 1500
end
copy running-config startup-config
```

Do not configure EIGRP on Internet.

If `passive-interface default` is rejected, use the same explicit fallback as
OSPF: R1 Fa0/0, R4 Fa0/0, and R5 Fa1/0 passive; internal links active.

Screenshot checkpoints:

- `screenshots/eigrp/01-r1-eigrp-config.png`
- `screenshots/eigrp/02-r2-eigrp-config.png`
- `screenshots/eigrp/03-r3-eigrp-config.png`
- `screenshots/eigrp/04-r4-eigrp-config.png`
- `screenshots/eigrp/05-r5-eigrp-config.png`

## C2. Verify EIGRP convergence and Internet isolation

On R1 and R4:

```text
show ip eigrp neighbors
show ip route eigrp
show ip route 0.0.0.0
```

On R5:

```text
show ip eigrp neighbors
show ip eigrp interfaces
show ip route 0.0.0.0
```

R5 Fa1/0 must not appear in `show ip eigrp interfaces`. R1/R4 should learn an
external EIGRP default route, normally marked `D*EX`.

Repeat the Internet checks, PC1 pings, and R5 NAT checks from B2.

Screenshot checkpoints:

- `screenshots/eigrp/06-r1-neighbors-and-routes.png`
- `screenshots/eigrp/07-r4-neighbors-and-routes.png`
- `screenshots/eigrp/08-r5-external-interface-excluded.png`
- `screenshots/eigrp/09-internet-still-isolated.png`
- `screenshots/eigrp/10-pc1-internal-and-internet-ping.png`
- `screenshots/eigrp/11-r5-nat-proof.png`

---

# Phase D — configure RIPv2

Open `KiarashShojaei-6-2-5-RIP.pkt`. Use RIPv2, not RIPv1.

RIP's `network` command is classful in this IOS, so `network 10.0.0.0` enables
RIP on the matching `10.*` interfaces. It does not include the external
`213.80.11.0/24` link.

## D1. RIPv2 configuration

### R1

```text
enable
configure terminal
router rip
 version 2
 no auto-summary
 network 10.0.0.0
 passive-interface default
 no passive-interface FastEthernet0/1
 no passive-interface FastEthernet1/0
end
copy running-config startup-config
```

### R2

```text
enable
configure terminal
router rip
 version 2
 no auto-summary
 network 10.0.0.0
 passive-interface default
 no passive-interface FastEthernet0/0
 no passive-interface FastEthernet0/1
end
copy running-config startup-config
```

### R3

```text
enable
configure terminal
router rip
 version 2
 no auto-summary
 network 10.0.0.0
 passive-interface default
 no passive-interface FastEthernet0/0
 no passive-interface FastEthernet0/1
end
copy running-config startup-config
```

### R4

```text
enable
configure terminal
router rip
 version 2
 no auto-summary
 network 10.0.0.0
 passive-interface default
 no passive-interface FastEthernet0/1
 no passive-interface FastEthernet1/0
end
copy running-config startup-config
```

### R5

```text
enable
configure terminal
router rip
 version 2
 no auto-summary
 network 10.0.0.0
 passive-interface default
 no passive-interface FastEthernet0/0
 no passive-interface FastEthernet0/1
 default-information originate
end
copy running-config startup-config
```

Do not configure RIP on Internet.

If `passive-interface default` is rejected, use the explicit passive-interface
fallback described in B1.

Screenshot checkpoints:

- `screenshots/rip/01-r1-rip-config.png`
- `screenshots/rip/02-r2-rip-config.png`
- `screenshots/rip/03-r3-rip-config.png`
- `screenshots/rip/04-r4-rip-config.png`
- `screenshots/rip/05-r5-rip-config.png`

## D2. Verify RIP convergence and Internet isolation

Wait at least 35 seconds for a complete periodic update, then run on R1 and R4:

```text
show ip protocols
show ip route rip
show ip route 0.0.0.0
show ip rip database
```

On R5, run:

```text
show ip protocols
show ip route 0.0.0.0
```

The external network must not be listed as a RIP network, and Fa1/0 must remain
passive/excluded. R1/R4 should learn a RIP default route marked `R*`.

Repeat the Internet, PC1, and NAT checks from B2.

Screenshot checkpoints:

- `screenshots/rip/06-r1-routes-and-database.png`
- `screenshots/rip/07-r4-routes-and-database.png`
- `screenshots/rip/08-r5-external-interface-excluded.png`
- `screenshots/rip/09-internet-still-isolated.png`
- `screenshots/rip/10-pc1-internal-and-internet-ping.png`
- `screenshots/rip/11-r5-nat-proof.png`

---

# Phase E — inspect routing packets in Simulation mode

Perform this phase inside each protocol's own `.pkt` file. Never mix protocols
in one file.

## E1. Prepare the Simulation view

For each file:

1. Click `Simulation` at the bottom right.
2. Click `Edit Filters`.
3. Click `Show None`, then enable only the current routing protocol:
   `OSPF`, `EIGRP`, or `RIP`.
4. Close the filter window.
5. Click `Reset Simulation` or clear the Event List.
6. Click `Auto Capture / Play`.
7. For OSPF/EIGRP, wait for a periodic Hello. For RIP, wait up to 30 seconds
   for a Response/update.
8. Click a colored envelope or the Event List's PDU square.
9. Open `Inbound PDU Details` or `Outbound PDU Details`.
10. Expand both the IP header and the routing-protocol header.

If an update packet does not appear, trigger one without leaving a link down:

```text
enable
configure terminal
interface FastEthernet0/1
 shutdown
 no shutdown
end
```

Run that on R2 while Simulation mode is active, then wait until adjacency and
routing recover.

## E2. OSPF packets to capture and fields to record

Capture one OSPF Hello and one Link State Update if available.

Record from the actual PDU:

- outer IP source and destination;
- IP protocol number 89;
- OSPF version and packet type;
- packet length and checksum;
- Router ID;
- Area ID, which must be 0;
- authentication field/type;
- for Hello: network mask, Hello interval, dead interval, priority, DR, BDR,
  and neighbor list;
- for Link State Update: LSA count, LSA type, link-state ID, advertising
  router, sequence number, age, and metric where shown.

Screenshot checkpoints:

- `screenshots/ospf/12-simulation-hello-packet.png`
- `screenshots/ospf/13-simulation-link-state-update.png`

## E3. EIGRP packets to capture and fields to record

Capture one EIGRP Hello and one EIGRP Update if available.

Record:

- outer IP source and destination;
- IP protocol number 88 and multicast destination `224.0.0.10` when shown;
- EIGRP version;
- opcode, such as Hello or Update;
- checksum, flags, sequence number, and acknowledgment number;
- autonomous-system number 100;
- parameter/route TLV type and length;
- K values and hold time in a Hello;
- destination prefix, next hop, minimum bandwidth, cumulative delay,
  reliability, load, MTU, and hop count in an Update.

Screenshot checkpoints:

- `screenshots/eigrp/12-simulation-hello-packet.png`
- `screenshots/eigrp/13-simulation-update-packet.png`

## E4. RIPv2 packets to capture and fields to record

Capture one RIPv2 Response that contains route entries.

Record:

- outer IP source and multicast destination `224.0.0.9` when shown;
- UDP source and destination port 520;
- RIP command, normally Response;
- RIP version 2;
- address-family identifier;
- route tag;
- advertised IPv4 network;
- subnet mask;
- next hop;
- metric/hop count.

If one update contains several route entries, explain that the route-entry
structure repeats for every advertised prefix.

Screenshot checkpoints:

- `screenshots/rip/12-simulation-response-header.png`
- `screenshots/rip/13-simulation-route-entries.png`

Return to `Realtime` after each packet inspection and wait for convergence
before continuing.

---

# Phase F — fair latency and route-choice comparison

Run the following sequence in all three files. Use identical commands, packet
sizes, endpoints, and link values. Perform one complete protocol at a time and
write every measured value into the tables near the end of this walkthrough.

Replace `<proto>` in screenshot names with exactly `ospf`, `eigrp`, or `rip`.

## F1. Baseline route and ping measurement

1. Confirm R1 Fa1/0 and R5 Fa0/0 still show:

   ```text
   BW 10000 Kbit
   DLY 10000 usec
   ```

2. On PC1, warm up ARP:

   ```text
   ping -n 5 10.10.16.1
   ```

3. Run and record the actual minimum, maximum, average, and packet-loss values:

   ```text
   ping -n 50 -l 1000 10.10.16.1
   ```

4. If Packet Tracer rejects `-l`, use `ping -n 50 10.10.16.1` and record that
   the installed PC command omitted the size option.
5. On PC1 run:

   ```text
   tracert 10.10.16.1
   ```

6. On R1 run:

   ```text
   show ip route 10.10.16.0
   ```

With equal link settings, all three protocols are expected to prefer the
two-hop lower path R1-R5-R4. Verify rather than assume.

Screenshot checkpoints:

- `screenshots/<proto>/14-baseline-ping-pc1-to-pc3.png`
- `screenshots/<proto>/15-baseline-tracert-pc1-to-pc3.png`
- `screenshots/<proto>/16-baseline-r1-route-to-pc3-lan.png`

## F2. Degrade the R1-R5 link

Apply the same severe values at both ends of only the R1-R5 link.

On R1:

```text
enable
configure terminal
interface FastEthernet1/0
 bandwidth 64
 delay 20000
end
```

On R5:

```text
enable
configure terminal
interface FastEthernet0/0
 bandwidth 64
 delay 20000
end
```

This advertises 64 kbit/s and 200 ms to protocols. Wait 45 seconds so even the
RIP project has time to send an update.

Verify both ends with `show interfaces`.

Screenshot checkpoint:

- `screenshots/<proto>/17-r1-r5-link-degraded.png`

## F3. Prove the protocol reaction

Repeat on R1 and PC1:

```text
show ip route 10.10.16.0
```

```text
tracert 10.10.16.1
ping -n 50 -l 1000 10.10.16.1
```

Expected result to verify:

| Protocol | Expected next hop from R1 after degradation | Reason |
|---|---|---|
| OSPF | R2, `10.10.8.2` | the very low advertised bandwidth raises OSPF cost on the lower path |
| EIGRP | R2, `10.10.8.2` | its composite metric uses minimum bandwidth and cumulative delay by default |
| RIPv2 | R5, `10.10.11.2` | lower path is still two hops; RIP ignores bandwidth and delay |

Screenshot checkpoints:

- `screenshots/<proto>/18-degraded-r1-route-to-pc3-lan.png`
- `screenshots/<proto>/19-degraded-tracert-pc1-to-pc3.png`
- `screenshots/<proto>/20-degraded-ping-pc1-to-pc3.png`

For one extra path proof, switch briefly to Simulation mode, filter to ICMP,
send one Simple PDU from PC1 to PC3, and use `Capture/Forward` until it arrives.
The topology animation must visibly show the selected upper or lower route.

Screenshot checkpoint:

- `screenshots/<proto>/21-degraded-icmp-path-in-simulation.png`

If OSPF or EIGRP still uses R5, first confirm both degraded interface values,
then confirm no old static route exists. Do not compensate with an unrelated
static route.

## F4. Generate simultaneous congestion traffic

Keep the R1-R5 link degraded for this step.

1. Return to Realtime mode.
2. Open PC1 and PC3 `Desktop` -> `Command Prompt` windows side by side.
3. On PC1 start:

   ```text
   ping -n 1000 -l 1400 10.10.16.1
   ```

4. Immediately start this on PC3:

   ```text
   ping -n 1000 -l 1400 10.10.6.1
   ```

5. If `-l` is unsupported, remove only `-l 1400`; keep `-n 1000`.
6. Let both commands finish. Record sent, received, lost, loss percentage,
   minimum, maximum, and average for each direction.
7. During the run, switch briefly to Simulation, filter only ICMP, and observe
   roughly 20 events. Then return to Realtime so the 1000-packet runs can
   finish. Do not try to single-step all 2000 requests.
8. Run `show interfaces` on the interfaces along the chosen path and record any
   input/output queue drops shown by Packet Tracer.

Screenshot checkpoints:

- `screenshots/<proto>/22-two-simultaneous-ping-streams.png`
- `screenshots/<proto>/23-pc1-1000-ping-result.png`
- `screenshots/<proto>/24-pc3-1000-ping-result.png`
- `screenshots/<proto>/25-congestion-simulation-events.png`
- `screenshots/<proto>/26-congestion-interface-counters.png`

Packet Tracer may show zero loss even under this synthetic test. Zero is a
valid measured result; never invent drops. The route choice and protocol
reaction remain independently provable with the route table, tracert, and
Simulation path.

## F5. Restore R1-R5 before Questions 7-9

On R1 Fa1/0 and R5 Fa0/0 restore:

```text
bandwidth 10000
delay 1000
```

Wait for convergence, then confirm R1 again uses R5 for
`10.10.16.0/24` under the normal baseline.

Screenshot checkpoint:

- `screenshots/<proto>/27-r1-r5-link-restored.png`

Save the Packet Tracer file.

---

# Phase G — Question 7 bonus: half through R2, half through R5

## G1. What must be true

There are two separate mechanisms:

1. The routing protocol must install two usable routes for
   `10.10.16.0/24`, one through R2 and one through R5.
2. The forwarding plane must distribute packets over those two installed
   routes.

Default CEF forwarding is normally per-destination/per-flow. Therefore one
PC1-to-PC3 flow can stay entirely on one route even when two equal routes are
installed. An exact alternating 50/50 result for the same source/destination
pair requires per-packet load sharing, which can reorder packets. Packet Tracer
7.0 does not faithfully model this physical-router behavior, so do not claim
that an animation proves exact 50/50 forwarding.

OSPF and RIP install multiple paths only when their route metrics are equal.
EIGRP can also install unequal-cost feasible paths with `variance`, but
`traffic-share balanced` distributes in proportion to metrics; unequal metrics
do not guarantee half and half. Exact half requires two equal metrics plus a
per-packet forwarding method.

## G2. Temporary Packet Tracer route-installation demonstration

Do this only as a temporary test. Take the screenshots, then undo it before
Phase H.

### OSPF file

With the Phase A baseline, each internal link has OSPF cost 10. The upper route
has one more such link than the lower route. On R1, make only the R1-R5 outgoing
cost 20:

```text
enable
configure terminal
interface FastEthernet1/0
 ip ospf cost 20
exit
router ospf 1
 maximum-paths 2
end
show ip route 10.10.16.0
```

The target evidence is two equal OSPF next hops: `10.10.8.2` and
`10.10.11.2`.

Screenshot checkpoint:

- `screenshots/ospf/28-q7-two-equal-routes-on-r1.png`

### EIGRP file

The lower route has one fewer 10 ms link. Add that missing 10 ms only to R1's
outgoing R1-R5 interface:

```text
enable
configure terminal
interface FastEthernet1/0
 delay 2000
exit
router eigrp 100
 variance 1
 maximum-paths 2
 traffic-share balanced
end
show ip route 10.10.16.0
show ip eigrp topology 10.10.16.0 255.255.255.0
```

The target is two equal feasible next hops. If only one appears, keep the
actual result and topology output; do not raise `variance` and call unequal
sharing 50/50.

Screenshot checkpoints:

- `screenshots/eigrp/28-q7-equal-route-attempt-on-r1.png`
- `screenshots/eigrp/29-q7-eigrp-topology-details.png`

### RIP file

RIP sees three hops through R2 and two through R5. Add one to the metric learned
from R5 for the PC3 LAN:

```text
enable
configure terminal
access-list 7 permit 10.10.16.0 0.0.0.255
router rip
 offset-list 7 in 1 FastEthernet1/0
 maximum-paths 2
end
show ip route 10.10.16.0
```

The target is two metric-3 RIP next hops.

Screenshot checkpoint:

- `screenshots/rip/28-q7-two-equal-routes-on-r1.png`

If this Packet Tracer IOS rejects `offset-list`, capture the error as
`screenshots/rip/28-q7-offset-list-not-supported.png` and use the real-IOS
explanation below. Do not replace RIP with static routes and present that as
RIP load balancing.

## G3. Real physical Cisco router method

After two equal routes are visible on R1, a physical IOS router that supports
CEF per-packet sharing can use:

```text
enable
configure terminal
ip cef
interface FastEthernet0/1
 ip load-sharing per-packet
exit
interface FastEthernet1/0
 ip load-sharing per-packet
end
show cef interface FastEthernet0/1
show cef interface FastEthernet1/0
```

The routing-process `maximum-paths 2` command limits installation to the two
desired paths. The metric equalization is protocol-specific as shown above.

On real hardware, verify with route/CEF counters and many packets, not one
`traceroute`. Per-packet sharing gives the closest interpretation of half and
half for one flow but risks packet reordering, so production networks normally
prefer per-flow/per-destination ECMP unless exact packet alternation is truly
required.

Official sources to cite when answering the bonus:

- [Cisco: CEF per-destination and per-packet load sharing](https://www.cisco.com/c/en/us/support/docs/ip/express-forwarding-cef/18285-loadbal-cef.html)
- [Cisco: EIGRP variance and traffic-share balanced](https://www.cisco.com/c/en/us/support/docs/ip/enhanced-interior-gateway-routing-protocol-eigrp/13677-19.html)
- [Cisco: RIP offset-list command](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_rip/command/irr-cr-book/irr-cr-rip.html)
- [Cisco: routing-protocol maximum paths and forwarding load balancing](https://www.cisco.com/c/en/us/support/docs/ip/border-gateway-protocol-bgp/5212-46.html)

## G4. Undo the temporary Question 7 changes

### OSPF R1

```text
enable
configure terminal
interface FastEthernet1/0
 no ip ospf cost
exit
router ospf 1
 no maximum-paths
end
```

### EIGRP R1

```text
enable
configure terminal
interface FastEthernet1/0
 delay 1000
exit
router eigrp 100
 no variance
 no maximum-paths
end
```

### RIP R1

```text
enable
configure terminal
router rip
 no offset-list 7 in 1 FastEthernet1/0
 no maximum-paths
exit
no access-list 7
end
```

If this IOS requires the configured value in the `no maximum-paths` command,
use `no maximum-paths 2`. If any `ip load-sharing per-packet` command was
accepted during a test, restore `ip load-sharing per-destination` on both R1
outgoing interfaces.

In each file, verify that R1 has returned to one route through R5:

```text
show ip route 10.10.16.0
```

Screenshot checkpoints:

- `screenshots/ospf/29-q7-temporary-settings-removed.png`
- `screenshots/eigrp/30-q7-temporary-settings-removed.png`
- `screenshots/rip/29-q7-temporary-settings-removed.png`

---

# Phase H — Question 8: make R4 send non-connected traffic only to R3

Interpret this requirement at the routing layer: directly connected traffic
cannot be sent to R3, but every remotely learned internal route and the default
route should prefer R3. R5 remains a less-preferred backup for Phase I.

## H1. OSPF: raise R4's cost toward R5

On R4 in the OSPF file:

```text
enable
configure terminal
interface FastEthernet1/0
 ip ospf cost 1000
end
```

## H2. EIGRP: worsen R4's metric toward R5

On R4 in the EIGRP file:

```text
enable
configure terminal
interface FastEthernet1/0
 bandwidth 64
 delay 20000
end
```

## H3. RIP: add an inbound metric offset to every route from R5

On R4 in the RIP file:

```text
enable
configure terminal
access-list 8 permit any
router rip
 offset-list 8 in 4 FastEthernet1/0
end
```

The offset makes R5-learned routes worse while keeping them below RIP's
unreachable metric 16 in this small topology. A passive interface is not a
replacement: in RIP it suppresses sent updates but can still allow received
updates, so it does not reliably enforce the desired next hop.

## H4. Verify Question 8 in each file

Wait for convergence. On R4 run:

```text
show ip route
show ip route 10.10.6.0
show ip route 0.0.0.0
```

All non-connected routes for which an alternate exists should use R3 at
`10.10.10.1`. Connected networks naturally remain connected and are not sent
to any next-hop router.

From PC3 run:

```text
tracert 10.10.6.1
ping -n 20 10.10.6.1
tracert 213.80.11.5
ping -n 20 213.80.11.5
```

The first routed hop after R4 should be R3. The Internet test should still
eventually reach R5 and then Internet, and NAT should still allow replies.

Screenshot checkpoints:

- `screenshots/<proto>/31-q8-r4-all-remote-routes-via-r3.png`
- `screenshots/<proto>/32-q8-pc3-to-pc1-via-r3.png`
- `screenshots/<proto>/33-q8-pc3-to-internet-via-r3.png`

For EIGRP, numbering follows its existing files, so use `31`, `32`, and `33`.
For RIP and OSPF use the same numbers even if one earlier number is unused.

Do not remove these Question 8 settings yet; they create the active upper path
needed for a fair R2 failure test.

---

# Phase I — Question 9: turn off R2 and measure convergence

Use R2, not R3, in all three files. R2 is one hop beyond R4's next hop, so R3
must communicate the failure through the routing protocol. This better exposes
protocol behavior than shutting R4's directly connected R3 interface.

## I1. Prepare a fair run

For each protocol file:

1. Confirm R2 is powered on and every link is green.
2. Wait at least 60 seconds after the last configuration change.
3. On R4, confirm `10.10.6.0/24` currently uses R3 (`10.10.10.1`).
4. From PC3, run two successful warm-up pings to PC1.
5. Keep a stopwatch ready. Use the same method for all protocols.

For EIGRP only, run this before the failure:

```text
show ip eigrp topology 10.10.6.0 255.255.255.0
```

Record whether the R5 route is shown as a feasible successor. This explains a
near-immediate DUAL switchover if observed.

Screenshot checkpoints:

- `screenshots/<proto>/34-q9-route-before-r2-failure.png`
- `screenshots/eigrp/34a-q9-feasible-successor-before-failure.png`

## I2. Measure live interruption

1. On PC3 start:

   ```text
   ping -n 1000 -w 1000 10.10.6.1
   ```

2. After at least five successful replies, open R2.
3. Click R2's `Physical` tab.
4. Start the stopwatch and turn R2's power switch off.
5. Watch the PC3 command output. Count consecutive timeouts.
6. Stop the timer when the first stable reply returns through the backup path.
7. Press `Ctrl+C` after at least ten recovered replies.
8. Record:

   - protocol;
   - failure time;
   - first recovered-reply time;
   - elapsed convergence time;
   - consecutive timeouts;
   - sent, received, and lost totals;
   - whether recovery was immediate, triggered, or timer-dependent.

If `-w` is unsupported, omit it and use only the stopwatch. Do not convert the
number of timeouts into seconds unless the displayed/used timeout was one
second.

Screenshot checkpoints:

- `screenshots/<proto>/35-q9-ping-interruption-after-r2-off.png`
- `screenshots/<proto>/36-q9-ping-recovered-through-backup.png`

## I3. Prove the new path after failure

While R2 is still off, on R4 run:

```text
show ip route 10.10.6.0
show ip route 0.0.0.0
```

From PC3 run:

```text
tracert 10.10.6.1
```

The recovered route should now use R5 at `10.10.12.2`.

Screenshot checkpoints:

- `screenshots/<proto>/37-q9-r4-backup-route-via-r5.png`
- `screenshots/<proto>/38-q9-tracert-via-r5.png`

## I4. Capture the protocol's failure reaction

For stronger proof, repeat a short version in Simulation mode:

1. Turn R2 back on and wait for complete convergence.
2. Confirm R4 has returned to R3.
3. Switch to Simulation and select only the current protocol plus ICMP.
4. Clear the Event List.
5. Start `ping -n 20 10.10.6.1` from PC3.
6. Turn R2 off again.
7. Use `Capture/Forward` or `Auto Capture/Play` until the control update and
   recovered ICMP path appear.
8. Open the routing packet that reports or reacts to the topology change.

Screenshot checkpoint:

- `screenshots/<proto>/39-q9-routing-update-after-failure.png`

For OSPF, look for a new/flooded LSA and SPF-driven route change. For EIGRP,
look for Update/Query/Reply behavior and DUAL successor selection. For RIP,
look for a triggered Response with a changed metric; if none appears
immediately, wait for the periodic update and record the wait.

## I5. Restore the final submitted state

1. Return to Realtime.
2. Turn R2 back on.
3. Wait until every link is green and all adjacencies/routes have converged.
4. Confirm R4 again uses R3 for `10.10.6.0/24`.
5. Confirm PC1-PC3 and PC3-Internet pings succeed.
6. Confirm Internet still has no internal route and no dynamic protocol.
7. Save the `.pkt` file.

Screenshot checkpoints:

- `screenshots/<proto>/40-q9-r2-restored-and-route-recovered.png`
- `screenshots/<proto>/41-final-end-to-end-pings.png`
- `screenshots/<proto>/42-final-internet-isolation.png`

Never submit a file with R2 powered off or with the temporary Question 7
equalization still active.

## I6. Compare the measured convergence results

Use actual observations. The expected theoretical tendency is:

- EIGRP can be fastest when DUAL already has a feasible successor.
- OSPF floods the changed link state and reruns SPF, normally converging
  quickly.
- RIP can be slower because it is distance-vector and uses periodic timers,
  although triggered updates may make this small Packet Tracer topology react
  faster than a full 30-second interval.

Do not write “EIGRP was fastest” unless your measured results show that. If two
protocols tie at Packet Tracer's timing resolution, report the tie and explain
the theoretical mechanisms separately.

---

# Measurement worksheet

Fill this while running the lab. These cells are deliberately blank; measured
values must come from your own files.

## Delay and route-change results

| Measurement | OSPF | EIGRP | RIPv2 |
|---|---:|---:|---:|
| Baseline next hop from R1 |  |  |  |
| Baseline path from `tracert` |  |  |  |
| Baseline ping minimum |  |  |  |
| Baseline ping maximum |  |  |  |
| Baseline ping average |  |  |  |
| Baseline ping loss |  |  |  |
| Next hop after R1-R5 degradation |  |  |  |
| Path after degradation |  |  |  |
| Degraded ping minimum |  |  |  |
| Degraded ping maximum |  |  |  |
| Degraded ping average |  |  |  |
| Degraded ping loss |  |  |  |
| PC1 1000-ping loss |  |  |  |
| PC3 1000-ping loss |  |  |  |
| Interface drops/counter change |  |  |  |

## Failure/convergence results

| Measurement | OSPF | EIGRP | RIPv2 |
|---|---:|---:|---:|
| R4 next hop before R2 failure |  |  |  |
| R4 next hop after R2 failure |  |  |  |
| Consecutive ping timeouts |  |  |  |
| Measured recovery time |  |  |  |
| Control packet observed |  |  |  |
| R4 next hop after R2 restoration |  |  |  |

## Routing-packet fields

| Protocol and packet type | Source/destination | Transport/IP protocol | Important header fields seen | Route data seen |
|---|---|---|---|---|
| OSPF Hello |  |  |  |  |
| OSPF Link State Update |  |  |  |  |
| EIGRP Hello |  |  |  |  |
| EIGRP Update |  |  |  |  |
| RIPv2 Response |  |  |  |  |

---

# Final completeness check

Before considering Experiment 1 complete, verify every item:

- [ ] Only Experiment 1 was built.
- [ ] The topology matches Figure 1 and the group-6 address table.
- [ ] OSPF, EIGRP, and RIP are in three separate `.pkt` files.
- [ ] No static internal routes remain from CNL4.
- [ ] R5 has the only static default route, toward `213.80.11.5`.
- [ ] R5 originates/redistributes that default inside each protocol file.
- [ ] The external R5 Fa1/0 link runs no OSPF, EIGRP, or RIP.
- [ ] Internet runs no dynamic protocol and has no `10.10.*` route.
- [ ] NAT proves reply traffic without teaching Internet internal routes.
- [ ] OSPF Hello and Link State Update fields were captured.
- [ ] EIGRP Hello and Update fields were captured.
- [ ] RIPv2 Response and route-entry fields were captured.
- [ ] Baseline pings used identical settings in all files.
- [ ] R1-R5 was degraded identically in all files.
- [ ] OSPF and EIGRP rerouting, and RIP's hop-count behavior, were proven with
      route table plus `tracert`/Simulation evidence.
- [ ] Two simultaneous 1000-packet streams were completed in every file.
- [ ] Question 7 includes the Packet Tracer limitation, equal-route
      requirements, CEF per-packet theory, real IOS commands, and sources.
- [ ] R4 was proven to prefer R3 in all three files.
- [ ] R2 was powered off in all three files and actual recovery was measured.
- [ ] R2 was restored before saving.
- [ ] All final links are green and end-to-end tests pass.
- [ ] All named screenshots exist and are readable.
- [ ] No `report.txt`, `report.tex`, Experiment 2 file, or final ZIP was created
      during this walkthrough stage.
