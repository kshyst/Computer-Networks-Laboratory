# CNL5 Question 9 — R2 failure and recovery fix

This guide corrects the Question 9 procedure for Packet Tracer 8.2.1. Apply it
separately to the OSPF, EIGRP, and RIP files.

The test traffic is from PC3 (`10.10.16.1`) to PC1 (`10.10.6.1`). Testing this
internal destination keeps NAT problems out of the routing-convergence result.

## What Question 9 must demonstrate

Before the failure, Question 8 must make R4 prefer R3 (`10.10.10.1`) while
leaving R5 (`10.10.12.2`) available as a worse backup. After R2 is powered off,
R4 must eventually replace the route through R3 with a route through R5.

Do not implement Question 8 by shutting R4 Fa1/0, making the R4-R5 routing
interface passive, filtering all routes from R5, or removing the protocol from
that link. Any of those actions removes the backup required by Question 9.

Expected paths:

```text
Before failure: PC3 -> R4 -> R3 -> R2 -> R1 -> PC1
After failure:  PC3 -> R4 -> R5 -> R1 -> PC1
```

## 1. Common physical checks before every run

R2 must initially be powered on. All router-to-router links must be green.

On R4, verify the direct R5 link:

```text
enable
ping 10.10.12.2
```

On R5, verify the direct R1 link:

```text
enable
ping 10.10.11.1
```

Both pings must succeed. If the first fails, enable both ends of R4-R5:

```text
configure terminal
interface FastEthernet1/0
no shutdown
end
```

```text
configure terminal
interface FastEthernet0/1
no shutdown
end
```

If the second fails, enable both ends of R5-R1:

```text
configure terminal
interface FastEthernet0/0
no shutdown
end
```

```text
configure terminal
interface FastEthernet1/0
no shutdown
end
```

R5 must also retain the external static default used by all three protocols:

```text
show ip route 0.0.0.0
```

If it is absent, restore it:

```text
configure terminal
ip route 0.0.0.0 0.0.0.0 213.80.11.5
end
```

Do not run OSPF, EIGRP, or RIP on the `213.80.11.0/24` Internet link.

Screenshot:

- `screenshots/<proto>/q9-01-r4-r5-and-r5-r1-pings.png`
- `screenshots/<proto>/q9-01a-r5-static-default.png`

Replace `<proto>` with `ospf`, `eigrp`, or `rip`.

## 2. Correctly power off R2

1. On PC3, open `Desktop` -> `Command Prompt`.
2. Start:

   ```text
   ping -n 1000 10.10.6.1
   ```

3. Wait for at least five successful replies.
4. Click R2 in the topology and open its `Physical` tab.
5. Find the small `I/O` switch on the router chassis. Zoom or scroll right if
   it is not visible.
6. Click that switch once to turn off the complete router.

Only the R1-R2 and R2-R3 links should become red. R4-R5 and R5-R1 must stay
green. Do not use an interface `shutdown`, delete R2, or click Packet Tracer's
global lightning-bolt power-cycle control.

Screenshots:

- `screenshots/<proto>/q9-02-r2-powered-off.png`
- `screenshots/<proto>/q9-03-ping-interruption.png`

---

# OSPF fix and test

## OSPF 1. Keep R5 as a real backup

Question 8 should make R5 expensive, not unavailable. Keep this on R4:

```text
configure terminal
interface FastEthernet1/0
ip ospf cost 1000
no shutdown
exit
router ospf 1
network 10.10.0.0 0.0.255.255 area 0
no passive-interface FastEthernet0/1
no passive-interface FastEthernet1/0
end
```

Repair the corresponding R5 interfaces if necessary:

```text
configure terminal
router ospf 1
network 10.10.0.0 0.0.255.255 area 0
no passive-interface FastEthernet0/0
no passive-interface FastEthernet0/1
default-information originate
end
```

Ensure R1 also exchanges OSPF with R5:

```text
configure terminal
router ospf 1
network 10.10.0.0 0.0.255.255 area 0
no passive-interface FastEthernet1/0
end
```

These `network` statements match only `10.10.*` interfaces, so OSPF remains
disabled on R5's external `213.80.11.4/24` interface.

## OSPF 2. Pre-failure proof

With R2 powered on, run:

```text
show ip ospf neighbor
show ip route 10.10.6.0
show ip ospf neighbor
show ip route 10.10.6.0
```

Required state:

- R4 has a `FULL` OSPF neighbor with R5 router ID `5.5.5.5` on Fa1/0.
- R5 has `FULL` neighbors with R4 and R1.
- R4's active PC1 route uses R3 at `10.10.10.1`.
- R5 knows the PC1 LAN through R1 at `10.10.11.1`.

Do not start the timed failure test until all four conditions are true.

Screenshot:

- `screenshots/ospf/q9-04-ospf-backup-ready-before-failure.png`

## OSPF 3. Expected recovery after R2 is off

OSPF should react within seconds after Packet Tracer marks the R2 links down.
On R4, run:

```text
show ip ospf neighbor
show ip route 10.10.6.0
show ip route 0.0.0.0
```

The R5 adjacency must remain `FULL`, and both remote routes must use
`10.10.12.2`. The route codes should be `O` for the PC1 LAN and `O*E2` for the
default route. A high cost of 1000 does not disable R5; OSPF uses that path when
the better path through R2 disappears.

From PC3, verify:

```text
tracert 10.10.6.1
ping -n 20 10.10.6.1
```

Screenshots:

- `screenshots/ospf/q9-05-ospf-ping-recovered.png`
- `screenshots/ospf/q9-06-ospf-route-via-r5.png`
- `screenshots/ospf/q9-07-ospf-tracert-via-r5.png`

If R4 can ping `10.10.12.2` but does not list R5 as a `FULL` neighbor, repair
the OSPF `network`, area, and `no passive-interface` settings above. If the
neighbor is `FULL` but the PC1 route is absent, inspect the route on R5 and the
R1-R5 adjacency instead of changing R4's cost.

---

# EIGRP fix and test

## EIGRP 1. Keep R5 as a real backup

Question 8 may worsen the metric on R4 Fa1/0, but that interface must remain in
EIGRP:

```text
configure terminal
interface FastEthernet1/0
bandwidth 64
delay 20000
no shutdown
exit
router eigrp 100
no auto-summary
network 10.10.0.0 0.0.255.255
no passive-interface FastEthernet0/1
no passive-interface FastEthernet1/0
end
```

Repair R5 and R1 if necessary:

```text
configure terminal
router eigrp 100
no auto-summary
network 10.10.0.0 0.0.255.255
no passive-interface FastEthernet0/0
no passive-interface FastEthernet0/1
redistribute static metric 100000 1000 255 1 1500
end
```

```text
configure terminal
router eigrp 100
no auto-summary
network 10.10.0.0 0.0.255.255
no passive-interface FastEthernet1/0
end
```

## EIGRP 2. Pre-failure proof

With R2 powered on, run:

```text
show ip eigrp neighbors
show ip route 10.10.6.0
show ip eigrp topology 10.10.6.0 255.255.255.0
show ip eigrp neighbors
show ip route 10.10.6.0
```

Required state:

- R4 lists R5 at `10.10.12.2` on Fa1/0.
- R5 lists R4 and R1 as neighbors.
- R4's active route uses R3 at `10.10.10.1`.
- R5 knows PC1 through R1 at `10.10.11.1`.

The topology output may or may not identify R5 as a feasible successor. If it
is not a feasible successor, EIGRP must send queries after R2 fails; this can
cause several timeouts but does not mean the backup is broken.

Screenshot:

- `screenshots/eigrp/q9-04-eigrp-backup-ready-before-failure.png`

## EIGRP 3. Expected recovery after R2 is off

Wait several seconds, then run on R4:

```text
show ip eigrp neighbors
show ip eigrp topology active
show ip route 10.10.6.0
show ip route 0.0.0.0
```

R5 must remain an EIGRP neighbor. After DUAL finishes, the PC1 route must be
`D` through `10.10.12.2`, and the default must be `D*EX` through the same next
hop. `show ip eigrp topology active` should eventually contain no active route.

From PC3, verify:

```text
tracert 10.10.6.1
ping -n 20 10.10.6.1
```

Screenshots:

- `screenshots/eigrp/q9-05-eigrp-ping-recovered.png`
- `screenshots/eigrp/q9-06-eigrp-route-via-r5.png`
- `screenshots/eigrp/q9-07-eigrp-tracert-via-r5.png`

If the R5 neighbor is absent, repair the EIGRP `network`, AS number 100, and
`no passive-interface` settings above. If the neighbor exists but R5 has no PC1
route, repair the R1-R5 EIGRP adjacency. Do not remove the Question 8 bandwidth
and delay merely to force a successful screenshot; they are what make R5 the
backup before the failure.

---

# RIP fix and test

## RIP 1. Keep R5 as a reachable, worse route

Question 8 should add four hops to routes learned from R5. It must not make the
R4-R5 interface passive or set the resulting metric to RIP's unreachable value
of 16.

On R4:

```text
configure terminal
access-list 8 permit any
router rip
version 2
no auto-summary
network 10.0.0.0
no passive-interface FastEthernet0/1
no passive-interface FastEthernet1/0
offset-list 8 in 4 FastEthernet1/0
end
```

On R5 and R1:

```text
configure terminal
router rip
version 2
no auto-summary
network 10.0.0.0
no passive-interface FastEthernet0/0
no passive-interface FastEthernet0/1
default-information originate
end
```

```text
configure terminal
router rip
version 2
no auto-summary
network 10.0.0.0
no passive-interface FastEthernet1/0
end
```

## RIP 2. Pre-failure proof

RIP has no neighbor-state command. With R2 powered on, run:

```text
show ip protocols
show ip route 10.10.6.0
show ip protocols
show ip route 10.10.6.0
```

Required state:

- R4 lists `10.10.12.2` as a routing-information source.
- R4's active PC1 route uses R3 at `10.10.10.1`.
- R5 knows PC1 through R1 at `10.10.11.1`.
- The offset is exactly 4, not a value that raises the backup metric to 16.

Wait at least one 30-second RIP update interval after the final configuration
change before beginning the timed failure test.

Screenshot:

- `screenshots/rip/q9-04-rip-backup-ready-before-failure.png`

## RIP 3. Expected recovery after R2 is off

RIP can take much longer than OSPF or EIGRP. A worse route from R5 may not be
accepted until RIP's invalid or hold-down processing finishes. Leave R2 off and
allow up to approximately 180 seconds before declaring failure. Record the
actual time and packet loss; do not shorten the measured convergence by clearing
the routing table.

After recovery, run:

```text
show ip protocols
show ip route 10.10.6.0
show ip route 0.0.0.0
```

The PC1 route must be `R` through `10.10.12.2`, and the default must be `R*`
through that same next hop. Record the displayed RIP metrics instead of assuming
a value.

From PC3, verify:

```text
tracert 10.10.6.1
ping -n 20 10.10.6.1
```

Screenshots:

- `screenshots/rip/q9-05-rip-ping-recovered.png`
- `screenshots/rip/q9-06-rip-route-via-r5.png`
- `screenshots/rip/q9-07-rip-tracert-via-r5.png`

If no R5 route appears after the full wait:

1. Confirm `ping 10.10.12.2` succeeds.
2. Confirm `show ip protocols` lists `10.10.12.2` as a source.
3. Confirm `show ip route 10.10.6.0` uses `10.10.11.1`.
4. Recheck that R4's offset is 4 and that Fa1/0 is not passive.

---

# Restore R2 after each protocol test

1. Open R2's `Physical` tab.
2. Click the same `I/O` switch to power it on.
3. Wait until both R2 links are green.
4. Wait for protocol convergence: several seconds for OSPF/EIGRP and at least
   one full update interval for RIP.
5. Confirm R4 again prefers R3:

   ```text
   show ip route 10.10.6.0
   show ip route 0.0.0.0
   ```

6. Confirm PC3 reaches PC1 through R4-R3-R2-R1:

   ```text
   tracert 10.10.6.1
   ping -n 20 10.10.6.1
   ```

7. Save the `.pkt` file with R2 powered on.

Screenshots:

- `screenshots/<proto>/q9-08-r2-powered-on-and-links-green.png`
- `screenshots/<proto>/q9-09-primary-route-restored-via-r3.png`

# Result checklist

- [ ] R2 was powered off from its Physical tab in all three files.
- [ ] R4-R5 and R5-R1 remained physically up during the failure.
- [ ] R5 was a valid protocol neighbor/source before the failure.
- [ ] Before failure, R4 used R3 for PC1 and the default route.
- [ ] During failure, the PC3 ping showed the real interruption.
- [ ] After convergence, R4 used R5 at `10.10.12.2`.
- [ ] The actual convergence time and packet loss were recorded separately.
- [ ] R2 was restored and the primary path through R3 returned before saving.
