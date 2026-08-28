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

- Packet Tracer 8.2.1 is installed.
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

2. If the simulated IOS rejects the pipe, run `show running-config` and scroll
   to the global `ip route` lines.
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

First verify the interface names on R5. In the transcripts below, type only the
text after each prompt; the prompt itself shows the IOS mode you must be in.

```text
R5> enable
R5# show ip interface brief
```

The expected assignments are:

- Fa0/0: `10.10.11.2`, internal;
- Fa0/1: `10.10.12.2`, internal;
- Fa1/0: `213.80.11.4`, external.

All three must show `up/up`. If the external address is on a different
interface, substitute that actual interface everywhere Fa1/0 appears below.

Enter the following commands on R5 exactly:

```text
R5> enable
R5# configure terminal
R5(config)# no access-list 1
R5(config)# access-list 1 permit 10.10.0.0 0.0.255.255
R5(config)# interface FastEthernet0/0
R5(config-if)# ip nat inside
R5(config-if)# exit
R5(config)# interface FastEthernet0/1
R5(config-if)# ip nat inside
R5(config-if)# exit
R5(config)# interface FastEthernet1/0
R5(config-if)# ip nat outside
R5(config-if)# exit
R5(config)# ip nat inside source list 1 interface FastEthernet1/0 overload
R5(config)# ip route 0.0.0.0 0.0.0.0 213.80.11.5
R5(config)# end
R5# copy running-config startup-config
```

Press Enter when IOS asks for the destination filename. Packet Tracer 8.2.1
normally prints nothing after a valid configuration command; returning to the
next prompt with no error means the command was accepted.

The prompt is essential:

- `ip nat inside` and `ip nat outside` must be entered at `R5(config-if)#`.
- Entering `ip nat inside` at `R5(config)#` produces `% Incomplete command`
  because IOS interprets it as the beginning of the global NAT command.
- The longer `ip nat inside source ... overload` command must be entered at
  `R5(config)#`, after leaving interface mode with `exit`.

If `no access-list 1` reports that ACL 1 does not exist, continue; its purpose
is only to remove an old or incorrect ACL before recreating it.

Now verify the saved configuration on R5:

```text
R5# show running-config
R5# show ip route 0.0.0.0
R5# show ip nat statistics
```

Press Space at every `--More--` prompt and confirm all of these lines are
present in `show running-config`:

```text
access-list 1 permit 10.10.0.0 0.0.255.255
ip nat inside source list 1 interface FastEthernet1/0 overload
```

The Fa0/0 and Fa0/1 sections must each contain `ip nat inside`; the Fa1/0
section must contain `ip nat outside`. At this point an empty translation
table or zero counters are normal because no internal PC has generated traffic
through R5 yet. The NAT lines in `show running-config`, not a nonempty table,
are the configuration proof.

If a NAT line is absent, repeat that interface step and check the prompt before
entering it. An `% Incomplete command` response to `ip nat inside` means you
are still at `R5(config)#`; first enter `interface FastEthernet0/0` or
`interface FastEthernet0/1` and confirm the prompt changes to
`R5(config-if)#`. Do not confuse missing running-config lines with an empty
translation table: the former is a configuration failure; the latter is
normal before traffic.

Do not configure any default or internal static route on the Internet router.
Verify there:

```text
show running-config
show ip route
show ip protocols
```

Expected Internet route-table state: only its connected/local
`213.80.11.0/24` entries; no `10.10.*` route and no routing protocol. If a
CNL4 return route is still present, remove the exact line shown in its running
configuration. For example, remove the old group summary with:

```text
enable
configure terminal
no ip route 10.10.0.0 255.255.0.0 213.80.11.4
end
copy running-config startup-config
```

Only run that `no ip route` command when the matching old route actually
exists. A successful PC-to-Internet ping while R5 has no NAT configuration is
evidence that an old return route probably remains on Internet; it is not NAT
proof.

Screenshot checkpoints:

- `screenshots/common/14a-r5-interface-map-before-nat.png`
- `screenshots/common/15-r5-nat-running-config.png`
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

If the simulated IOS rejects `passive-interface default`, remove that line and
explicitly make only these interfaces passive: R1 Fa0/0, R4 Fa0/0, and R5
Fa1/0. Keep the same internal interfaces active.

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

From PC1, warm up ARP and verify the internal destination:

```text
ping 10.10.16.1
```

From PC1 generate enough external traffic for the translation to remain easy
to inspect:

```text
ping -n 100 213.80.11.5
```

Immediately afterward, run on R5:

```text
show running-config
show ip nat translations
show ip nat statistics
show access-lists
```

If the two `show ip nat` commands still print no entries in Packet Tracer but
the ping succeeds, do not add a return route to Internet. Use Simulation mode
and inspect the outbound ICMP packet as it leaves R5 Fa1/0. Its source must be
translated from PC1's `10.10.6.1` to R5's `213.80.11.4`. Capture that packet
detail together with the NAT lines in `show running-config`. This is stronger
proof than an unimplemented display command.

If the source remains `10.10.6.1`, NAT is not operating. Recheck the three
interface roles and the global overload line; do not claim the successful ping
as NAT evidence.

Screenshot checkpoints:

- `screenshots/ospf/06-r1-neighbors-and-routes.png`
- `screenshots/ospf/07-r4-neighbors-and-routes.png`
- `screenshots/ospf/08-r5-external-interface-excluded.png`
- `screenshots/ospf/09-internet-still-isolated.png`
- `screenshots/ospf/10-pc1-internal-and-internet-ping.png`
- `screenshots/ospf/11-r5-nat-proof.png`

Do not continue until PC1-to-PC3 works, the default route exists internally,
PC1-to-Internet replies succeed through NAT, and Internet remains unaware of
the internal prefixes.

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

If `passive-interface default` is rejected, remove that line and the associated
`no passive-interface` lines. Under `router eigrp 100`, configure R1 Fa0/0, R4
Fa0/0, and R5 Fa1/0 as explicit passive interfaces. R2 and R3 have only
internal router links and need no passive interface in this fallback.

```text
R1(config-router)# passive-interface FastEthernet0/0
R4(config-router)# passive-interface FastEthernet0/0
R5(config-router)# passive-interface FastEthernet1/0
```

Screenshot checkpoints:

- `screenshots/eigrp/01-r1-eigrp-config.png`
- `screenshots/eigrp/02-r2-eigrp-config.png`
- `screenshots/eigrp/03-r3-eigrp-config.png`
- `screenshots/eigrp/04-r4-eigrp-config.png`
- `screenshots/eigrp/05-r5-eigrp-config.png`

For each screenshot, run `show running-config` and show the complete
`router eigrp 100` block, including AS 100, the network statement, and the
active/passive interface choices. Include R5's redistribution metric in its
screenshot.

## C2. Verify EIGRP convergence and Internet isolation

What to expect: EIGRP sends Hellos to form adjacencies on the internal
router-to-router interfaces, exchanges bounded route Updates, and uses DUAL to
select a successor. With the equal baseline links, bandwidth and delay make
the shorter R1-R5-R4 path preferable. R5 redistributes its static default, so
other routers see that default as an external EIGRP route. Passive PC-facing
interfaces advertise their connected LANs but send no EIGRP Hellos.

Wait about 15 seconds after the last EIGRP command, then verify every layer in
the order below. Do not move to RIP until all four checks pass.

### C2.1 Verify EIGRP neighbors

On R1:

```text
show ip eigrp neighbors
```

R1 must have exactly these two internal neighbors:

| Neighbor address | Local interface | Device |
|---|---|---|
| `10.10.8.2` | Fa0/1 | R2 |
| `10.10.11.2` | Fa1/0 | R5 |

On R4:

```text
show ip eigrp neighbors
```

R4 must have exactly these two internal neighbors:

| Neighbor address | Local interface | Device |
|---|---|---|
| `10.10.10.1` | Fa0/1 | R3 |
| `10.10.12.2` | Fa1/0 | R5 |

On R5:

```text
show ip eigrp neighbors
```

R5 must have R1 (`10.10.11.1`) on Fa0/0 and R4 (`10.10.12.1`) on Fa0/1.
There must never be a neighbor on R5 Fa1/0.

Also verify the two middle routers:

```text
R2# show ip eigrp neighbors
R3# show ip eigrp neighbors
```

R2 must list R1 `10.10.8.1` on Fa0/0 and R3 `10.10.9.2` on Fa0/1. R3 must
list R2 `10.10.9.1` on Fa0/0 and R4 `10.10.10.2` on Fa0/1. This proves every
one of the five internal router-to-router links participates correctly.

In `show ip eigrp neighbors`, `Hold` counts down and is refreshed by Hellos,
`Uptime` should keep increasing, `SRTT` and `RTO` are EIGRP reliability timers
rather than ping delay, and `Q Cnt` should normally be 0 after convergence.

If a neighbor is missing, check these items on both ends of that link before
changing anything else:

```text
show ip interface brief
show running-config
show ip protocols
```

The interfaces must be `up/up`, both routers must use EIGRP AS 100, the
`network 10.10.0.0 0.0.255.255` command must exist, and the router-to-router
interface must be listed after `no passive-interface`.

### C2.2 Verify EIGRP routes and the redistributed default

On R1:

```text
show ip route eigrp
show ip route 10.10.16.0
show ip route 0.0.0.0
show ip eigrp topology 10.10.16.0 255.255.255.0
```

Expected baseline results:

- the PC3 LAN `10.10.16.0/24` is marked `D` and uses R5 at `10.10.11.2`;
- the default route is marked `D*EX` and uses R5 at `10.10.11.2`;
- `D` means an internal EIGRP route, `EX` means an external route, and `*`
  marks a candidate default route.

The topology entry should be in state `P` (Passive), meaning the route is
stable and not currently being recomputed. Record its successor count,
feasible distance, and reported distance; `Passive` here is a DUAL state and
is unrelated to the `passive-interface` command.

On R4:

```text
show ip route eigrp
show ip route 10.10.6.0
show ip route 0.0.0.0
```

The PC1 LAN `10.10.6.0/24` must be a `D` route through R5 at
`10.10.12.2`. The default must be `D*EX` through the same next hop.

On R5:

```text
show ip route 0.0.0.0
show running-config
```

R5 itself must retain the static default `S* 0.0.0.0/0` through
`213.80.11.5`. If R1/R4 have no `D*EX` default, check that R5 contains both
the static default and this exact EIGRP command:

```text
redistribute static metric 100000 1000 255 1 1500
```

### C2.3 Prove the external interface is excluded

On R5:

```text
show ip protocols
show ip eigrp interfaces
```

Confirm AS 100 and the `10.10.0.0/16` network are shown. Fa0/0 and Fa0/1 are
the only EIGRP router links; Fa1/0 and `213.80.11.0/24` must not appear as an
active EIGRP interface/network.

On Internet:

```text
show ip protocols
show ip route
```

Expected result: no routing protocol, no `10.10.*` route, and only the
connected/local `213.80.11.0/24` entries.

### C2.4 Prove end-to-end forwarding and NAT

From PC1, run each test separately:

```text
ping 10.10.6.2
ping 10.10.11.2
ping -n 20 10.10.16.1
tracert 10.10.16.1
ping -n 100 213.80.11.5
tracert 213.80.11.5
```

Expected results:

- the first two pings prove PC1 reaches its gateway and R5;
- PC1-to-PC3 succeeds and normally uses R1-R5-R4 at baseline;
- the Internet trace reaches R5 at `10.10.11.2` and then Internet;
- a trace that reaches `10.10.11.2` and then times out points to R5 NAT, not
  EIGRP.

Immediately after the 100-packet Internet ping, run on R5:

```text
show ip nat statistics
show access-lists
show ip nat translations
```

The statistics must list Fa0/0 and Fa0/1 as inside and Fa1/0 as outside. ACL 1
must show matches, and a translation must map PC1 `10.10.6.1` to R5
`213.80.11.4`. If ACL matches stay at zero, recheck the NAT interface roles. If
ACL matches increase but there is no translation, recheck the global
`ip nat inside source list 1 interface FastEthernet1/0 overload` command.

Save only after every expected result is present:

```text
copy running-config startup-config
```

Screenshot checkpoints:

- `screenshots/eigrp/06-r1-neighbors-routes-and-default.png`
- `screenshots/eigrp/06a-r2-and-r3-neighbors.png`
- `screenshots/eigrp/07-r4-neighbors-routes-and-default.png`
- `screenshots/eigrp/08-r5-default-and-external-interface-excluded.png`
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

If `passive-interface default` is rejected, remove that line and the matching
`no passive-interface` lines. Configure only the PC-facing interfaces as
passive on R1 and R4; R5 Fa1/0 is outside `network 10.0.0.0` but may also be
made explicitly passive:

```text
R1(config-router)# passive-interface FastEthernet0/0
R4(config-router)# passive-interface FastEthernet0/0
R5(config-router)# passive-interface FastEthernet1/0
```

R2 and R3 have only internal router links, so neither needs an explicitly
passive interface in this fallback.

Screenshot checkpoints:

- `screenshots/rip/01-r1-rip-config.png`
- `screenshots/rip/02-r2-rip-config.png`
- `screenshots/rip/03-r3-rip-config.png`
- `screenshots/rip/04-r4-rip-config.png`
- `screenshots/rip/05-r5-rip-config.png`

For each screenshot, run `show running-config` and display the complete
`router rip` block. On R5, include `default-information originate`; on every
router, make the active and passive interfaces readable.

## D2. Verify RIP convergence and Internet isolation

Wait at least 35 seconds for a complete periodic update. RIP has no neighbor
table, so use `show ip protocols`, its routing information sources, and the
route table as the proof.

### D2.1 Verify RIPv2 operation on R1 and R4

On R1:

```text
show ip protocols
show ip route rip
show ip route 10.10.16.0
show ip route 0.0.0.0
show ip rip database
```

Confirm all of the following:

- the protocol is RIP and sends/receives version 2;
- automatic summarization is disabled;
- `10.0.0.0` is the only configured RIP network;
- Fa0/0, the PC1 LAN, is passive;
- the routing information sources include R2 `10.10.8.2` and R5
  `10.10.11.2` after updates arrive;
- `10.10.16.0/24` is marked `R` through R5 at `10.10.11.2` with the lower
  baseline hop count;
- the default is marked `R*` through R5 at `10.10.11.2`.

On R4:

```text
show ip protocols
show ip route rip
show ip route 10.10.6.0
show ip route 0.0.0.0
show ip rip database
```

R4's sources must include R3 `10.10.10.1` and R5 `10.10.12.2`. The PC1 LAN
and default route must both prefer R5 at `10.10.12.2` in the baseline.

On R2 and R3, run:

```text
show ip protocols
show ip route rip
```

R2's routing sources must include R1 `10.10.8.1` and R3 `10.10.9.2`.
R3's sources must include R2 `10.10.9.1` and R4 `10.10.10.2`. Together with
the R1/R4 checks, this proves RIP updates cross every internal link.

### D2.2 Verify R5's default and external-link exclusion

On R5:

```text
show ip protocols
show ip route 0.0.0.0
show running-config
```

Confirm that R5 retains `S* 0.0.0.0/0` through `213.80.11.5`, the RIP block
contains `default-information originate`, and `213.80.11.0/24` is not a RIP
network. Its routing sources must include R1 `10.10.11.1` and R4
`10.10.12.1`. Fa1/0 must remain excluded from RIP updates.

On Internet:

```text
show ip protocols
show ip route
```

Internet must have no RIP process and no `10.10.*` route. Its only network is
the directly connected `213.80.11.0/24` LAN.

### D2.3 Verify forwarding and NAT

From PC1:

```text
ping 10.10.6.2
ping 10.10.11.2
ping -n 20 10.10.16.1
tracert 10.10.16.1
ping -n 100 213.80.11.5
tracert 213.80.11.5
```

At baseline, PC1-to-PC3 should use R1-R5-R4 because RIP counts two hops on the
lower path and three on the upper path. The Internet test must succeed through
R5 NAT.

Immediately afterward, on R5:

```text
show ip nat statistics
show access-lists
show ip nat translations
```

Require NAT hits and a `10.10.6.1` to `213.80.11.4` translation. If RIP routes
are absent, wait another 35 seconds and recheck version 2, `network 10.0.0.0`,
and passive interfaces. If the default alone is absent, check R5's static
default plus `default-information originate`. If the trace reaches R5 and then
stops, troubleshoot NAT rather than changing RIP.

Save after all checks pass:

```text
copy running-config startup-config
```

Screenshot checkpoints:

- `screenshots/rip/06-r1-protocol-routes-and-default.png`
- `screenshots/rip/07-r4-protocol-routes-and-default.png`
- `screenshots/rip/07a-r2-and-r3-sources-and-routes.png`
- `screenshots/rip/08-r5-default-and-external-interface-excluded.png`
- `screenshots/rip/09-internet-still-isolated.png`
- `screenshots/rip/10-pc1-internal-and-internet-ping.png`
- `screenshots/rip/11-r5-nat-proof.png`

---

# Phase E — inspect routing packets in Simulation mode

Perform this phase inside each protocol's own `.pkt` file. Never mix protocols
in one file.

## E1. Prepare the Simulation view

Use the R2-R3 link for all three captures so the comparison uses the same
source link. R2 Fa0/1 is `10.10.9.1`; R3 Fa0/0 is `10.10.9.2`.

1. Save the current file before changing link state.
2. In Realtime, confirm PC1 can still ping PC3.
3. Click `Simulation` at the bottom right.
4. Open `Edit Filters`, click `Show None`, and enable only the current routing
   protocol: `OSPF`, `EIGRP`, or `RIP`.
5. Clear the Event List.
6. Click `Auto Capture / Play` and wait for a stable control packet: normally
   up to 10 seconds for OSPF, about 5 seconds for EIGRP, or up to 30 seconds for
   RIP.
7. Pause playback as soon as the needed packet appears. Click its colored
   square in the Event List, open `Inbound PDU Details` or
   `Outbound PDU Details`, and expand the IP and routing-protocol headers.

To force a topology-change packet, clear the Event List and shut only R2
Fa0/1. Do not enter `no shutdown` in the same command sequence; Packet Tracer
needs time to generate and display the failure event.

```text
R2> enable
R2# configure terminal
R2(config)# interface FastEthernet0/1
R2(config-if)# shutdown
R2(config-if)# end
```

Use `Capture/Forward` until the routing update caused by the shutdown is
visible. After capturing it, restore the link:

```text
R2# configure terminal
R2(config)# interface FastEthernet0/1
R2(config-if)# no shutdown
R2(config-if)# end
```

For EIGRP, the link restoration is useful because the new adjacency exchanges
an EIGRP Update. Continue Capture/Forward until that Update appears.

## E2. OSPF packets to capture and fields to record

Open the OSPF file and capture:

1. a stable Hello on the R2-R3 link;
2. a Link State Update generated by shutting or restoring R2 Fa0/1.

Record from the actual PDU:

- outer source `10.10.9.1` or `10.10.9.2` and the displayed destination,
  normally `224.0.0.5` for an OSPF multicast on this Ethernet link;
- IP protocol number 89;
- OSPF version 2 and packet type 1 for Hello or type 4 for Link State Update;
- packet length and checksum;
- Router ID, expected to be `2.2.2.2` for R2 or `3.3.3.3` for R3;
- Area ID, which must be 0;
- authentication field/type;
- for Hello: network mask, Hello interval, dead interval, priority, DR, BDR,
  and neighbor list;
- for Link State Update: LSA count, LSA type, link-state ID, advertising
  router, sequence number, age, and metric where shown.

Screenshot checkpoints:

- `screenshots/ospf/12-simulation-hello-packet.png`
- `screenshots/ospf/13-simulation-link-state-update.png`
- `screenshots/ospf/13a-simulation-link-restored.png`

## E3. EIGRP packets to capture and fields to record

Open the EIGRP file and capture:

1. a stable Hello on the R2-R3 link;
2. an Update exchanged when R2 Fa0/1 is restored and the R2-R3 adjacency
   forms again.

Record:

- outer source `10.10.9.1` or `10.10.9.2` and destination `224.0.0.10` for a
  normal multicast Hello; record the actual destination if an Update is
  unicast;
- IP protocol number 88;
- EIGRP version;
- opcode, normally 5 for Hello and 1 for Update;
- checksum, flags, sequence number, and acknowledgment number;
- autonomous-system number 100;
- parameter/route TLV type and length;
- K values and hold time in a Hello; normally K1 and K3 are 1 while K2, K4,
  and K5 are 0;
- destination prefix, next hop, minimum bandwidth, cumulative delay,
  reliability, load, MTU, and hop count in an Update.

Expected behavior: Hellos discover and maintain neighbors; Updates carry only
needed route information rather than a 30-second full table. If the Event List
shows Query or Reply during the shutdown, record it as additional convergence
evidence, but still capture the Update after restoration for this section.

Screenshot checkpoints:

- `screenshots/eigrp/12-simulation-hello-packet.png`
- `screenshots/eigrp/13-simulation-update-packet.png`
- `screenshots/eigrp/13a-simulation-link-restored.png`

## E4. RIPv2 packets to capture and fields to record

Open the RIP file and capture one RIPv2 Response containing route entries. A
periodic Response should appear within 30 seconds; shutting R2 Fa0/1 can also
produce a triggered Response with a changed or poisoned metric.

Record:

- outer source `10.10.9.1` or `10.10.9.2` and multicast destination
  `224.0.0.9`;
- IP protocol number 17 for UDP;
- UDP source and destination port 520;
- RIP command, normally 2 for Response;
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
- `screenshots/rip/13a-simulation-link-restored.png`

## E5. Prove the trigger link was restored

Return to Realtime after each protocol capture. Verify the R2-R3 link is
`up/up`:

```text
R2# show ip interface brief
R3# show ip interface brief
```

Then use the matching recovery command: `show ip ospf neighbor` in the OSPF
file, `show ip eigrp neighbors` in the EIGRP file, or `show ip route rip` in
the RIP file.

For RIP, wait at least 35 seconds before judging recovery. Finally, from PC1:

```text
ping -n 10 10.10.16.1
```

Do not continue until the ping succeeds and R2 Fa0/1 is no longer shut down.
Use the protocol's `13a-simulation-link-restored.png` screenshot to show the
`up/up` interface plus recovered neighbor or route output.

---

# Phase F — fair latency and route-choice comparison

Run the following sequence in all three files. Use identical commands, packet
sizes, endpoints, and link values. Perform one complete protocol at a time and
write every measured value into the tables near the end of this walkthrough.

Replace `<proto>` in screenshot names with exactly `ospf`, `eigrp`, or `rip`.

## F1. Baseline route and ping measurement

1. On R1 and R5, verify the lower-path link values:

   ```text
   R1# show interfaces FastEthernet1/0
   R5# show interfaces FastEthernet0/0
   ```

   Both outputs must show:

   ```text
   BW 10000 Kbit
   DLY 10000 usec
   ```

   `delay 1000` is displayed by IOS as `DLY 10000 usec` because the
   configuration unit is tens of microseconds.

2. On R1, record the baseline route:

   ```text
   show ip route 10.10.16.0
   ```

   Expected route code and next hop:

   | File | Route code | Baseline next hop |
   |---|---|---|
   | OSPF | `O` | R5, `10.10.11.2` |
   | EIGRP | `D` | R5, `10.10.11.2` |
   | RIP | `R` | R5, `10.10.11.2` |

3. On PC1, warm up ARP:

   ```text
   ping -n 5 10.10.16.1
   ```

4. Run and record the actual minimum, maximum, average, and packet-loss values:

   ```text
   ping -n 50 -l 1000 10.10.16.1
   ```

5. If Packet Tracer rejects `-l`, use `ping -n 50 10.10.16.1` and record that
   the installed PC command omitted the size option.
6. On PC1 run:

   ```text
   tracert 10.10.16.1
   ```

The trace should show the lower R1-R5-R4 path. Record the actual hop addresses
and measured ping values; do not replace `<1 ms` with an invented delay.

Screenshot checkpoints:

- `screenshots/<proto>/14-baseline-ping-pc1-to-pc3.png`
- `screenshots/<proto>/15-baseline-tracert-pc1-to-pc3.png`
- `screenshots/<proto>/16-baseline-r1-route-to-pc3-lan.png`

## F2. Degrade the R1-R5 link

Apply the same severe values at both ends of only the R1-R5 link.

On R1:

```text
R1> enable
R1# configure terminal
R1(config)# interface FastEthernet1/0
R1(config-if)# bandwidth 64
R1(config-if)# delay 20000
R1(config-if)# end
```

On R5:

```text
R5> enable
R5# configure terminal
R5(config)# interface FastEthernet0/0
R5(config-if)# bandwidth 64
R5(config-if)# delay 20000
R5(config-if)# end
```

This advertises 64 kbit/s and 200 ms to routing protocols. Wait until OSPF or
EIGRP recalculates; in the RIP file wait at least 35 seconds even though RIP
will ignore these metric inputs.

Verify both ends:

```text
R1# show interfaces FastEthernet1/0
R5# show interfaces FastEthernet0/0
```

Both must show `BW 64 Kbit` and `DLY 200000 usec`. If only one end changed,
correct it before comparing protocols.

Screenshot checkpoint:

- `screenshots/<proto>/17-r1-r5-link-degraded.png`

## F3. Prove the protocol reaction

On R1:

```text
show ip route 10.10.16.0
```

Then on PC1:

```text
tracert 10.10.16.1
ping -n 50 -l 1000 10.10.16.1
```

Expected result to verify:

| Protocol | Expected next hop from R1 after degradation | Reason |
|---|---|---|
| OSPF | `O` through R2, `10.10.8.2` | the very low advertised bandwidth raises OSPF cost on the lower path |
| EIGRP | `D` through R2, `10.10.8.2` | its composite metric uses minimum bandwidth and cumulative delay by default |
| RIPv2 | `R` through R5, `10.10.11.2` | the lower path is still two hops; RIP ignores bandwidth and delay |

Screenshot checkpoints:

- `screenshots/<proto>/18-degraded-r1-route-to-pc3-lan.png`
- `screenshots/<proto>/19-degraded-tracert-pc1-to-pc3.png`
- `screenshots/<proto>/20-degraded-ping-pc1-to-pc3.png`

For one extra path proof, switch briefly to Simulation mode, filter to ICMP,
send one Simple PDU from PC1 to PC3, and use `Capture/Forward` until it arrives.
The topology animation must visibly show the selected upper or lower route.

Screenshot checkpoint:

- `screenshots/<proto>/21-degraded-icmp-path-in-simulation.png`

If OSPF still uses R5, run `show ip ospf interface FastEthernet1/0` on R1 and
confirm the cost increased. If EIGRP still uses R5, run
`show ip eigrp topology 10.10.16.0 255.255.255.0` and confirm the R5 metric is
worse. In either file, also confirm both degraded interface values and remove
any old static route. Do not compensate with an unrelated static route.

## F4. Generate simultaneous congestion traffic

Keep the R1-R5 link degraded for this step.

1. Return to Realtime mode.
2. Before generating traffic, record interface counters with `show interfaces`
   on the expected path:

   - OSPF/EIGRP upper path: R1 Fa0/1 and R2 Fa0/0; R2 Fa0/1 and R3 Fa0/0;
     R3 Fa0/1 and R4 Fa0/1;
   - RIP lower path: R1 Fa1/0 and R5 Fa0/0; R5 Fa0/1 and R4 Fa1/0.

3. Open PC1 and PC3 `Desktop` -> `Command Prompt` windows side by side.
4. On PC1 start:

   ```text
   ping -n 1000 -l 1400 10.10.16.1
   ```

5. Immediately start this on PC3:

   ```text
   ping -n 1000 -l 1400 10.10.6.1
   ```

6. If `-l` is unsupported, remove only `-l 1400`; keep `-n 1000`.
7. Let both commands finish. Record sent, received, lost, loss percentage,
   minimum, maximum, and average for each direction.
8. Run the same `show interfaces` commands used in step 2 and calculate the
   counter changes. Record any input/output queue drops shown by Packet Tracer.
9. For readable Simulation evidence, clear the Event List after the long pings
   finish, filter only ICMP, then start `ping -n 20` from both PCs. Capture
   roughly 20 events and stop; do not put all 2000 long-ping packets into the
   Event List.

Screenshot checkpoints:

- `screenshots/<proto>/22-two-simultaneous-ping-streams.png`
- `screenshots/<proto>/22a-interface-counters-before-congestion.png`
- `screenshots/<proto>/23-pc1-1000-ping-result.png`
- `screenshots/<proto>/24-pc3-1000-ping-result.png`
- `screenshots/<proto>/25-congestion-simulation-events.png`
- `screenshots/<proto>/26-congestion-interface-counters.png`

The IOS `bandwidth` value primarily supplies a routing metric; it is not a
physical rate limiter on real hardware. Packet Tracer may therefore show zero
loss in this synthetic test. Zero is a valid measured result; never invent
drops. Route choice and protocol reaction remain independently provable with
the route table, tracert, and Simulation path.

## F5. Restore R1-R5 before Questions 7-9

On R1:

```text
R1# configure terminal
R1(config)# interface FastEthernet1/0
R1(config-if)# bandwidth 10000
R1(config-if)# delay 1000
R1(config-if)# end
```

On R5:

```text
R5# configure terminal
R5(config)# interface FastEthernet0/0
R5(config-if)# bandwidth 10000
R5(config-if)# delay 1000
R5(config-if)# end
```

Verify both interfaces again show `BW 10000 Kbit` and `DLY 10000 usec`. Wait
for convergence—at least 35 seconds in RIP—then run:

```text
R1# show ip route 10.10.16.0
```

The single baseline next hop must again be R5 at `10.10.11.2`. From PC1,
confirm `tracert 10.10.16.1` again uses R1-R5-R4.

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
8.2.1 does not faithfully model this physical-router behavior, as the professor
noted, so do not claim that an animation proves exact 50/50 forwarding.

OSPF and RIP install multiple paths only when their route metrics are equal.
EIGRP can also install unequal-cost feasible paths with `variance`, but
`traffic-share balanced` distributes in proportion to metrics; unequal metrics
do not guarantee half and half. Exact half requires two equal metrics plus a
per-packet forwarding method.

## G2. Temporary Packet Tracer route-installation demonstration

Do this only as a temporary test. Take the screenshots, then undo it before
Phase H. Before changing each file, run `show ip route 10.10.16.0` on R1 and
confirm Phase F was restored: there must be one baseline route through R5 at
`10.10.11.2`.

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
`10.10.11.2`. The route output must contain two `via` lines with the same OSPF
metric. If only R2 appears, the R1 Fa1/0 cost is too high; if only R5 appears,
it is too low. Confirm `show ip ospf interface FastEthernet1/0` reports cost
20 before changing any other value.

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
actual result and topology output. First confirm R1 Fa1/0 shows
`DLY 20000 usec`, wait for convergence, and check whether the topology output
reports two successors with the same feasible distance. Do not raise
`variance` and call unequal sharing 50/50.

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

After waiting at least 35 seconds, the target is two RIP next hops,
`10.10.8.2` and `10.10.11.2`, both with hop metric 3. Confirm the route output
contains two `via` lines with the same `[120/3]` value.

Screenshot checkpoint:

- `screenshots/rip/28-q7-two-equal-routes-on-r1.png`

If this Packet Tracer IOS rejects `offset-list`, capture the error as
`screenshots/rip/28-q7-offset-list-not-supported.png` and use the real-IOS
explanation below. Do not replace RIP with static routes and present that as
RIP load balancing.

For each protocol, run `ping -n 20 10.10.16.1` and several
`tracert 10.10.16.1` commands from PC1 after the two routes are installed.
Record what Packet Tracer actually forwards, but use the two equal `via`
entries—not an assumed animation pattern—as proof that the routing protocol
installed both paths. State explicitly that Packet Tracer 8.2.1 did not prove
exact alternating 50/50 per-packet forwarding.

## G3. Real physical Cisco router method

Do not enter this block as required Packet Tracer configuration. It is the
real-IOS answer requested by the professor for the feature Packet Tracer 8.2.1
does not model faithfully.

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

Useful verification commands on the physical R1 are:

```text
show ip route 10.10.16.0
show ip cef 10.10.16.0
show cef interface FastEthernet0/1
show cef interface FastEthernet1/0
```

The route table must show both next hops, and both outgoing interfaces must
show per-packet load sharing before claiming the requested behavior.

Official sources to cite when answering the bonus:

- [Cisco: CEF per-destination and per-packet load sharing](https://www.cisco.com/c/en/us/support/docs/ip/express-forwarding-cef/18285-loadbal-cef.html)
- [Cisco: EIGRP variance and traffic-share balanced](https://www.cisco.com/c/en/us/support/docs/ip/enhanced-interior-gateway-routing-protocol-eigrp/13677-19.html)
- [Cisco: RIP offset-list command](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_rip/command/irr-cr-book/irr-cr-rip.html)
- [Cisco: protocol-independent `maximum-paths` command](https://www.cisco.com/c/en/us/td/docs/ios/iproute_pi/command/reference/iri_book/iri_pi1.html)

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
 no traffic-share balanced
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

If any `ip load-sharing per-packet` command was accepted during a test, restore
`ip load-sharing per-destination` on both R1 outgoing interfaces.

In each file, verify that R1 has returned to one route through R5:

```text
show ip route 10.10.16.0
```

Also verify the protocol-specific temporary metric is gone:

- OSPF: `show ip ospf interface FastEthernet1/0` returns to the calculated
  baseline cost;
- EIGRP: `show interfaces FastEthernet1/0` shows `DLY 10000 usec`;
- RIP: `show running-config` contains no `offset-list 7` and no ACL 7.

From PC1, run `tracert 10.10.16.1`; it must again use R1-R5-R4 before Phase H.
Save each `.pkt` file after this verification so no temporary Question 7
configuration survives accidentally.

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
show ip ospf interface FastEthernet1/0
```

Confirm the output reports cost 1000. This makes the direct R4-to-R5 direction
far more expensive than R4-R3-R2-R1-R5.

## H2. EIGRP: worsen R4's metric toward R5

On R4 in the EIGRP file:

```text
enable
configure terminal
interface FastEthernet1/0
 bandwidth 64
 delay 20000
end
show interfaces FastEthernet1/0
```

Confirm `BW 64 Kbit` and `DLY 200000 usec`. EIGRP will advertise the resulting
worse route metric to its neighbors.

## H3. RIP: add an inbound metric offset to every route from R5

On R4 in the RIP file:

```text
enable
configure terminal
access-list 8 permit any
router rip
 offset-list 8 in 4 FastEthernet1/0
end
show running-config
```

The offset makes R5-learned routes worse while keeping them below RIP's
unreachable metric 16 in this small topology. A passive interface is not a
replacement: in RIP it suppresses sent updates but can still allow received
updates, so it does not reliably enforce the desired next hop.

## H4. Verify Question 8 in each file

Wait for convergence: about 15 seconds for OSPF/EIGRP and at least 35 seconds
for RIP. On R4 run:

```text
show ip route
show ip route 10.10.6.0
show ip route 10.10.11.0
show ip route 0.0.0.0
```

Expected R4 result:

| File | PC1 route code | Default code | Next hop for both |
|---|---|---|---|
| OSPF | `O` | `O*E2` | R3, `10.10.10.1` |
| EIGRP | `D` | `D*EX` | R3, `10.10.10.1` |
| RIP | `R` | `R*` | R3, `10.10.10.1` |

All remotely learned routes for which an alternate exists—including
`10.10.6.0/24`, `10.10.8.0/24`, `10.10.9.0/24`, `10.10.11.0/24`, and the
default—must use R3. R4's `10.10.10.0/24`, `10.10.12.0/24`, and
`10.10.16.0/24` networks remain connected and cannot be sent to a next hop.

Prove R3 will not send the default straight back to R4. On R3 run:

```text
show ip route 10.10.6.0
show ip route 0.0.0.0
```

R3 should use R2 at `10.10.9.1`, not R4 at `10.10.10.2`, for the default and
the upper path toward PC1. The worsened route R4 reports through R5 makes R2
the better choice. Do not continue if R3 points the default back to R4; that
would create a forwarding loop.

From PC3 run:

```text
tracert 10.10.6.1
ping -n 20 10.10.6.1
tracert 213.80.11.5
ping -n 20 213.80.11.5
```

The first routed hop after R4 should be R3. The Internet test should reach R5
through the complete R4-R3-R2-R1-R5 path and receive replies through NAT.

Screenshot checkpoints:

- `screenshots/<proto>/30a-q8-r4-metric-configuration.png`
- `screenshots/<proto>/31-q8-r4-all-remote-routes-via-r3.png`
- `screenshots/<proto>/31a-q8-r3-uses-r2-not-r4.png`
- `screenshots/<proto>/32-q8-pc3-to-pc1-via-r3.png`
- `screenshots/<proto>/33-q8-pc3-to-internet-via-r3.png`

Replace `<proto>` with `ospf`, `eigrp`, or `rip` in all five names.

Do not remove these Question 8 settings yet; they create the active upper path
needed for a fair R2 failure test.

On R4, run `copy running-config startup-config`, then save the `.pkt` file
before starting the failure test.

---

# Phase I — Question 9: turn off R2 and measure convergence

Use R2, not R3, in all three files. R2 is one hop beyond R4's next hop, so R3
must communicate the failure through the routing protocol. This better exposes
protocol behavior than shutting R4's directly connected R3 interface.

## I1. Prepare a fair run

For each protocol file:

1. Confirm R2 is powered on and every link is green.
2. On R2 and R3, run `show ip interface brief`; both ends of the R2-R3 link
   must be `up/up`.
3. Wait about 15 seconds for OSPF/EIGRP or at least 35 seconds for RIP after
   the last change.
4. On R4, run:

   ```text
   show ip route 10.10.6.0
   show ip route 0.0.0.0
   ```

   Both routes must currently use R3 at `10.10.10.1`.
5. On R3, run `show ip route 0.0.0.0`; it must use R2 at `10.10.9.1`, proving
   the active path will not loop back to R4.
6. Confirm protocol state:

   - OSPF: `show ip ospf neighbor` on R3 must show R2 in `FULL` state;
   - EIGRP: `show ip eigrp neighbors` on R3 must list R2 at `10.10.9.1`;
   - RIP: `show ip protocols` on R3 must list R2 as a routing information
     source.
7. From PC3, run `ping -n 10 10.10.6.1` and
   `tracert 10.10.6.1`. Both must succeed through R4-R3-R2-R1.
8. Keep a stopwatch ready and use the same timing method in all three files.

For EIGRP only, run this before the failure:

```text
show ip eigrp topology 10.10.6.0 255.255.255.0
```

Record whether the R5 route is shown as a feasible successor. This explains a
near-immediate DUAL switchover if observed. Record the successor count,
feasible distance, each next hop, and each reported distance; do not label the
R5 path a feasible successor unless the topology output actually includes it.

Screenshot checkpoints:

- `screenshots/<proto>/34-q9-route-before-r2-failure.png`
- `screenshots/eigrp/34a-q9-feasible-successor-before-failure.png`

## I2. Measure live interruption

1. Keep the PC3 Command Prompt and R2 Physical windows visible at the same
   time. On PC3 start:

   ```text
   ping -n 1000 -w 1000 10.10.6.1
   ```

2. After at least five successful replies, start the stopwatch and turn off
   R2 from its `Physical` tab.
3. Watch the PC3 command output. Count consecutive timeouts.
4. Stop the timer when the first stable reply returns through the backup path.
5. Press `Ctrl+C` after at least ten recovered replies.
6. Record:

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

Do not use different failure methods between files. Power off the complete R2
device in OSPF, EIGRP, and RIP; do not shut an interface in one file and power
off the router in another.

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
ping -n 20 10.10.6.1
tracert 213.80.11.5
ping -n 20 213.80.11.5
```

The recovered PC1 route and default route must now use R5 at `10.10.12.2`.
The route codes remain protocol-specific: `O`/`O*E2`, `D`/`D*EX`, or
`R`/`R*`. PC3-to-PC1 should follow R4-R5-R1, while PC3-to-Internet should
follow R4-R5-Internet and receive replies through NAT.

If recovery does not occur:

- OSPF: check `show ip ospf neighbor` and `show ip ospf database` on R3/R4;
- EIGRP: check `show ip eigrp topology active` and
  `show ip eigrp topology 10.10.6.0 255.255.255.0` on R4;
- RIP: wait at least 35 seconds, then check `show ip protocols` and
  `show ip route rip` on R4.

Do not restore R2 until the backup route and both traces are captured.

Screenshot checkpoints:

- `screenshots/<proto>/37-q9-r4-backup-route-via-r5.png`
- `screenshots/<proto>/38-q9-tracert-via-r5.png`
- `screenshots/<proto>/38a-q9-internet-through-r5.png`

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

Record the actual source, destination, packet type, changed prefix/metric, and
event time. A Hello packet alone is not proof of the failure reaction.

## I5. Restore the final submitted state

1. Return to Realtime.
2. Turn R2 back on.
3. On R2 and R3, run `show ip interface brief` and wait until the R2-R3 link is
   `up/up`.
4. Wait about 15 seconds for OSPF/EIGRP or at least 35 seconds for RIP.
5. Verify recovery with the matching command:

   - OSPF: `show ip ospf neighbor` on R2 must show R1 and R3 in `FULL` state;
   - EIGRP: `show ip eigrp neighbors` on R2 must list R1 and R3;
   - RIP: `show ip route rip` on R2 must again contain routes through both
     sides after a periodic update.
6. On R4, run `show ip route 10.10.6.0` and `show ip route 0.0.0.0`; both must
   again use R3 at `10.10.10.1`.
7. On R3, run `show ip route 0.0.0.0`; it must again use R2 at
   `10.10.9.1`.
8. Run these final data-plane tests:

   ```text
   PC1> ping -n 20 10.10.16.1
   PC3> ping -n 20 10.10.6.1
   PC3> ping -n 20 213.80.11.5
   ```

9. On Internet, run `show ip protocols` and `show ip route`; require no
   dynamic protocol and no `10.10.*` route.
10. On R5, run `show ip nat translations` after the Internet ping and retain
    the translation proof.
11. On R4, run `copy running-config startup-config` so the Question 8 setting
    remains in the submitted file.
12. Save the `.pkt` file.

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

For EIGRP, connect the explanation to the captured topology state: if R5 was a
feasible successor, DUAL can promote it without a network-wide recomputation;
if the route became Active, describe the Query/Reply exchange that completed
the new calculation.

Do not write “EIGRP was fastest” unless your measured results show that. If two
protocols tie at Packet Tracer's timing resolution, report the tie and explain
the theoretical mechanisms separately.

---

# Measurement worksheet

Fill this while running the lab. These cells are deliberately blank; measured
values must come from your own files.

## Protocol readiness results

| Verification | OSPF | EIGRP | RIPv2 |
|---|---|---|---|
| R1 internal neighbors or RIP sources |  |  |  |
| R4 internal neighbors or RIP sources |  |  |  |
| R1 route code/next hop to `10.10.16.0/24` |  |  |  |
| R4 route code/next hop to `10.10.6.0/24` |  |  |  |
| Learned default code and next hop |  |  |  |
| PC1-to-PC3 result |  |  |  |
| PC1-to-Internet result |  |  |  |
| NAT translation and ACL match proof |  |  |  |
| Internet has no protocol/internal route |  |  |  |

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

## Question 7 and Question 8 results

| Verification | OSPF | EIGRP | RIPv2 |
|---|---|---|---|
| Q7 R1 next hop through R2 |  |  |  |
| Q7 R1 next hop through R5 |  |  |  |
| Q7 equal metric shown |  |  |  |
| Q7 Packet Tracer forwarding actually observed |  |  |  |
| Q7 temporary configuration removed |  |  |  |
| Q8 R4 PC1 route code/next hop |  |  |  |
| Q8 R4 default code/next hop |  |  |  |
| Q8 R3 default next hop, proving no loop |  |  |  |
| Q8 PC3-to-PC1 trace |  |  |  |
| Q8 PC3-to-Internet trace |  |  |  |

## Failure/convergence results

| Measurement | OSPF | EIGRP | RIPv2 |
|---|---:|---:|---:|
| R4 next hop before R2 failure |  |  |  |
| R3 default next hop before R2 failure |  |  |  |
| EIGRP feasible successor present | N/A |  | N/A |
| R4 next hop after R2 failure |  |  |  |
| Consecutive ping timeouts |  |  |  |
| Measured recovery time |  |  |  |
| Control packet type observed |  |  |  |
| Changed prefix/metric in control packet |  |  |  |
| R4 next hop after R2 restoration |  |  |  |
| R3 default next hop after restoration |  |  |  |

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
- [ ] EIGRP shows the expected two-neighbor set on every internal router.
- [ ] R1/R4 learn EIGRP `D*EX` defaults and RIP `R*` defaults through R5.
- [ ] The external R5 Fa1/0 link runs no OSPF, EIGRP, or RIP.
- [ ] Internet runs no dynamic protocol and has no `10.10.*` route.
- [ ] NAT running-config plus a translated packet proves reply traffic without
      teaching Internet any internal route.
- [ ] OSPF Hello and Link State Update fields were captured.
- [ ] EIGRP Hello and Update fields were captured.
- [ ] RIPv2 Response and route-entry fields were captured.
- [ ] R2 Fa0/1 was restored after every Simulation packet-capture trigger.
- [ ] Baseline pings used identical settings in all files.
- [ ] R1-R5 was degraded identically in all files.
- [ ] OSPF and EIGRP rerouting, and RIP's hop-count behavior, were proven with
      route table plus `tracert`/Simulation evidence.
- [ ] Two simultaneous 1000-packet streams were completed in every file.
- [ ] Question 7 includes the Packet Tracer limitation, equal-route
      requirements, CEF per-packet theory, real IOS commands, and sources.
- [ ] Question 7's temporary costs, delays, offset lists, and maximum-paths
      settings were removed before Question 8.
- [ ] R4 was proven to prefer R3 in all three files, and R3 was proven to use
      R2 rather than return the default to R4.
- [ ] R2 was powered off in all three files and actual recovery was measured.
- [ ] The post-failure R4 route and traceroute both prove backup through R5.
- [ ] R2 was restored before saving, and its adjacencies/routes recovered.
- [ ] All final links are green and end-to-end tests pass.
- [ ] Every worksheet cell is filled from actual Packet Tracer output; no
      theoretical expectation is reported as a measurement.
- [ ] All named screenshots exist and are readable.
- [ ] No `report.txt`, `report.tex`, Experiment 2 file, or final ZIP was created
      during this walkthrough stage.
