# Experiment 2 — DHCP Snooping: standalone Packet Tracer walkthrough

Follow Steps 1–11 in order. Steps 1–10 use the two-switch topology from Figure 2 on page 6; Step 11 is a **separate routed interpretation** of the router-and-ACL requirement. Commands below contain no device prompts. Open the named device's CLI, press Enter, and start from user EXEC or privileged EXEC mode before each block; configuration blocks enter and leave configuration mode themselves. If a new router asks whether to enter the initial configuration dialog, answer `no`.

**Verification status:** this is a procedure and evidence plan, not an executed lab report. Packet Tracer was not available in the authoring environment (neither executable nor `/opt/pt` installation was found). No `.pkt`, screenshot, packet trace, lease, ACL hit count, or rate-limit event has been fabricated. Record your Packet Tracer version and switch/router models when you run the lab. If a command is rejected, preserve the exact error and report that simulator limitation instead of claiming the feature worked.

## Scope, addressing, and files

- The assignment's third-octet/group-number transformation is explicitly introduced under **Experiment 1**. For Experiment 2, retain the diagram's `10.0.0.0/24` addresses; do not silently add the group number to them. Ask the instructor if a broader transformation is intended.
- The diagram labels gateway `10.0.0.1`, but **does not show a device that owns it**. Keep `.1` as the pool's reference gateway in Steps 1–10. Same-subnet DHCP and local communication do not require a working gateway. A failed ping to `.1` or to an off-subnet DNS address is therefore not a DHCP failure. Do not add a router or assign `.1` to the Layer-2 switch merely to make that ping pass.
- The handout places an example `ip dhcp snooping trust` near Step 6 and labels some switch commands `R1`. Operational correction: run them on **SW1**, enable snooping globally **and for VLAN 10** in Step 6, and defer **all trust** until Step 7, as the numbered requirements demand.
- The source headers call the lab “Experiment Seven,” whereas the requested repository/submission identity is CNL8. Use the requested archive name **`FullName-6-2-8.zip`**, replacing `FullName` with your name. Do not silently change it to experiment 7.
- Save the switched experiment as **`E2-DHCP-SNOOPING.pkt`** and the separate routed experiment as **`E2-ROUTER-ACL.pkt`**. Optional baseline save: **`E2-BASELINE.pkt`** before Step 6. These are files you create while executing this guide, not files supplied by this document.
- Screenshot names below are exact basenames. For a checkpoint requiring several windows, use `-a`, `-b`, and `-c` before `.png`; these are frames of one checkpoint, not additional base checkpoints. Capture evidence immediately where requested.

## Step 1 — Build Figure 2 and configure the initial network

### 1.1 Place and cable devices

Place two **2960** switches named `SW1` and `SW2`, three **PC-PT** devices named `PC1`, `PC2`, and `PC3`, and two **Server-PT** devices named `LEGAL-DHCP` and `ROGUE-DHCP`. Set names through **Config → Display Name** if needed.

| Device endpoint | Switch endpoint | Initial connection | Role |
|---|---|---|---|
| LEGAL-DHCP FastEthernet0 | SW1 FastEthernet0/1 | Connected | Legal DHCP server |
| PC1 FastEthernet0 | SW1 FastEthernet0/2 | Connected | Main test client |
| SW1 FastEthernet0/23 | SW2 FastEthernet0/1 | Connected | VLAN 10 trunk |
| PC2 FastEthernet0 | SW2 FastEthernet0/2 | Connected | Downstream client |
| PC3 FastEthernet0 | SW2 FastEthernet0/3 | Connected | Downstream client |
| ROGUE-DHCP FastEthernet0 | SW1 FastEthernet0/24 | **Not connected yet** | Rogue server, attached in Steps 5 and 8 |

Use copper straight-through for endpoint-to-switch links and copper crossover for the switch-to-switch link, or Packet Tracer's automatic cable tool. The rogue server is placed and configured now but its cable is deliberately absent. Wait until connected links are green; a trunk can take time to start forwarding.

### 1.2 Configure VLANs and ports

**SW1 → CLI; start in EXEC mode:**

```text
enable
configure terminal
hostname SW1
vlan 10
name Users
exit
interface fastEthernet0/1
switchport mode access
switchport access vlan 10
spanning-tree portfast
no shutdown
exit
interface fastEthernet0/2
switchport mode access
switchport access vlan 10
spanning-tree portfast
no shutdown
exit
interface fastEthernet0/24
switchport mode access
switchport access vlan 10
spanning-tree portfast
no shutdown
exit
interface fastEthernet0/23
switchport mode trunk
switchport trunk allowed vlan 10
no shutdown
exit
no ip dhcp snooping
end
write memory
```

**SW2 → CLI; start in EXEC mode:**

```text
enable
configure terminal
hostname SW2
vlan 10
name Users
exit
interface fastEthernet0/1
switchport mode trunk
switchport trunk allowed vlan 10
no shutdown
exit
interface fastEthernet0/2
switchport mode access
switchport access vlan 10
spanning-tree portfast
no shutdown
exit
interface fastEthernet0/3
switchport mode access
switchport access vlan 10
spanning-tree portfast
no shutdown
exit
no ip dhcp snooping
end
write memory
```

Do not enter `switchport trunk encapsulation dot1q` on the fixed-802.1Q 2960. No management IP or SVI address is needed for this lab.

**SW1 → CLI; EXEC verification:**

```text
enable
show vlan brief
show interfaces trunk
```

**SW2 → CLI; EXEC verification:**

```text
enable
show vlan brief
show interfaces trunk
```

Check VLAN 10 `Users`, the named access ports, and VLAN 10 allowed/active/forwarding on SW1 Fa0/23 and SW2 Fa0/1. A trunk is not listed as a VLAN 10 access port in `show vlan brief`.

**Screenshot now — `E2-01-topology.png`:** capture the complete named topology, interface labels, VLAN/subnet note, and disconnected rogue server. Use the workspace's text-note tool to label `VLAN 10 Users — 10.0.0.0/24 — reference GW 10.0.0.1 (not implemented)`.

**Screenshot now — `E2-02-vlan-trunks.png`:** use frame `-a` for SW1 and `-b` for SW2; keep each switch's VLAN and trunk verification readable.

### 1.3 Manually configure Server-PT interfaces and DHCP pools

These are **configuration actions**, not just an addressing reference. On each server open **Desktop → IP Configuration**, choose **Static**, and enter:

| Field | LEGAL-DHCP | ROGUE-DHCP |
|---|---|---|
| IPv4 Address | `10.0.0.10` | `10.0.0.200` |
| Subnet Mask | `255.255.255.0` | `255.255.255.0` |
| Default Gateway | `10.0.0.1` | `10.0.0.1` |
| DNS Server | `10.0.0.10` | `203.0.113.53` |

On **each server → Services → DHCP**, select interface **FastEthernet0** if a selector exists, set **Service On**, and configure the following. Use **Add** for a new pool and **Save** when editing it; reselect the saved pool to confirm its values. Remove unrelated editable pools. If the built-in `serverPool` cannot be removed, edit that pool to the values shown rather than leaving an extra competing/default scope; record the retained name.

| DHCP GUI field | LEGAL-DHCP | ROGUE-DHCP |
|---|---|---|
| Pool Name | `LEGAL-USERS` | `ROGUE-USERS` |
| Default Gateway | `10.0.0.1` | `10.0.0.200` |
| DNS Server | `10.0.0.10` | `203.0.113.53` |
| Start IP Address | `10.0.0.100` | `10.0.0.150` |
| Subnet Mask | `255.255.255.0` | `255.255.255.0` |
| Maximum Number of Users | `40` | `40` |
| TFTP Server, if shown | `0.0.0.0` | `0.0.0.0` |
| WLC Address, if shown | `0.0.0.0` | `0.0.0.0` |

The legal range is `.100–.139`; the rogue range is `.150–.189`. Neither includes the server addresses or `.1`, and the ranges do not overlap. `203.0.113.53` is a deliberately fake/documentation DNS value, not a real external service. The rogue advertises itself as a gateway even though Server-PT does not route traffic. The legal DNS field identifies the legal configuration; a working DNS service is not required to demonstrate address acquisition.

**Screenshot now — `E2-03-legal-server.png`:** frame `-a` shows the legal server's static IP window; frame `-b` shows Service On and the saved DHCP pool values.

**Screenshot now — `E2-04-rogue-server.png`:** frame `-a` shows the rogue static IP; frame `-b` shows its saved DHCP pool and fake DNS/gateway. Keep the rogue cable absent.

On each PC, open **Desktop → IP Configuration → DHCP**, starting with PC1, then PC2, then PC3. Wait for acquisition. PC1 is likely to obtain `.100` when it is first, but use its **actual** lease in all observations; do not force a static `.100` address.

**Step 1 evidence:** `E2-01-topology.png`, `E2-02-vlan-trunks.png`, `E2-03-legal-server.png`, `E2-04-rogue-server.png`.

## Step 2 — Observe PC1's current IP address

**PC1 → Desktop → Command Prompt:**

```text
ipconfig /all
```

Record the actual IPv4 address, mask, gateway, DNS, and physical/MAC address. It should have a legal-pool lease at this initial stage. If this build's `ipconfig /all` omits DNS or server identity, use **Desktop → IP Configuration** for DNS and Step 4's DHCP packet for the server identifier. Do not infer a DHCP server merely from the default gateway.

**Screenshot now — `E2-05-current-ip.png`:** show PC1's current IP information before releasing or renewing it; use a second frame for any missing GUI fields.

**Step 2 evidence:** `E2-05-current-ip.png`.

## Step 3 — Renew PC1's DHCP lease

**PC1 → Desktop → Command Prompt:**

```text
ipconfig /renew
ipconfig /all
```

Renewing an existing lease can contact the previously selected server and retain the same IP. A changed address is not required to prove renewal. For a fresh Discover/Offer exchange in the next step, release first. If this build rejects a PC command, preserve the error, then use **Desktop → IP Configuration → Static**, clear the old IPv4 configuration, and select **DHCP** again; describe that GUI re-acquisition as a substitute, not as a successful rejected command.

**Screenshot now — `E2-06-renew.png`:** capture the renewal action and resulting IP configuration.

**Step 3 evidence:** `E2-06-renew.png`.

## Step 4 — Identify the server that gave PC1 its address

1. Enter **Simulation** mode at the bottom right.
2. Open **Edit Filters**, clear unrelated protocols, and enable **DHCP** (some releases label the relevant traffic DHCP/BOOTP). Include ARP if needed for troubleshooting.
3. Clear old events. On PC1's Command Prompt run the block below.
4. Use **Capture/Forward** to follow Discover, Offer, Request, and ACK. Click an event envelope, then **OSI Model**, **Inbound PDU Details**, or **Outbound PDU Details** as appropriate.
5. Inspect the legal server's Offer/ACK source address and DHCP **Server Identifier (Option 54)** if displayed. Correlate the client hardware address and transaction with PC1. If Option 54 is not decoded by your build, record the visible source device/IP and the selected Request/ACK exchange instead.

**PC1 → Desktop → Command Prompt, while Simulation is active:**

```text
ipconfig /release
ipconfig /renew
```

**Answer:** with the rogue disconnected, PC1 should select `LEGAL-DHCP`, address `10.0.0.10`. The lease range and DNS corroborate this, but the responding server and selected exchange are the decisive evidence. DHCP ordinarily uses client UDP port 68 and server UDP port 67. The initial sequence is Discover → Offer → Request → ACK; not every later renewal repeats a broadcast Discover. [1][2]

**Screenshot now — `E2-07-legal-dora.png`:** capture the event list and legal Offer/ACK details identifying `10.0.0.10`. Use `-a` for the event sequence and `-b` for packet details if necessary. Return to Realtime after completing the exchange.

**Step 4 evidence:** `E2-07-legal-dora.png`.

## Step 5 — Demonstrate when a rogue server can win

**Question:** when could the rogue answer PC1? On this shared VLAN, without snooping, both reachable DHCP servers can receive a fresh broadcast and offer a lease. A client may select the rogue's offer depending on arrival timing and implementation. It is not guaranteed that the rogue wins every race. A simple renew of a still-valid legal lease may contact only the legal server.

1. Keep snooping disabled. Connect **ROGUE-DHCP FastEthernet0 → SW1 Fa0/24** and wait for forwarding.
2. Enter Simulation mode with the DHCP filter. Release and renew PC1 using the following block. Look for offers from both servers. Record whichever server actually wins; do not repeatedly relabel a legal win as a rogue win.

**PC1 → Desktop → Command Prompt:**

```text
ipconfig /release
ipconfig /renew
ipconfig /all
```

3. To demonstrate rogue acquisition deterministically, return to Realtime, open **LEGAL-DHCP → Services → DHCP → Off**. Leave the legal server's NIC and saved pool unchanged. The rogue's DHCP service stays On.
4. Repeat the PC1 block in a clean Simulation event list. Capture the rogue's Offer and ACK, then finish the exchange and inspect PC1's configuration. Expect a `.150–.189` lease, gateway `10.0.0.200`, and fake DNS `203.0.113.53`. This models the legal server being unavailable; it is **not** proof that the rogue beat an active legal server on timing.

**Screenshot now — `E2-08-rogue-baseline.png`:** `-a` shows the rogue Offer/ACK from `10.0.0.200`; `-b` shows PC1's rogue lease and DNS; `-c` shows the legal service Off for the deterministic test. If both-server race evidence is available, retain it as another clearly labeled frame and state its actual winner.

**Mandatory cleanup before Step 6:** turn **LEGAL-DHCP DHCP On**, disconnect the rogue cable from **SW1 Fa0/24**, and release/renew PC1 in Realtime. Verify a legal lease and legal DNS again. Leave PC2/PC3 connected. Save `E2-BASELINE.pkt` if desired.

**Screenshot now — `E2-09-baseline-restored.png`:** show the disconnected rogue and legal service On (`-a`), then PC1's restored legal configuration (`-b`).

**Step 5 evidence:** `E2-08-rogue-baseline.png`, `E2-09-baseline-restored.png`.

## Step 6 — Enable snooping on SW1 with no trusted ports

Start from the restored baseline. The rogue is disconnected and the legal server is On.

**SW1 → CLI; start in EXEC mode:**

```text
enable
configure terminal
ip dhcp snooping
ip dhcp snooping vlan 10
interface fastEthernet0/1
no ip dhcp snooping trust
exit
interface fastEthernet0/2
no ip dhcp snooping trust
exit
interface fastEthernet0/23
no ip dhcp snooping trust
exit
interface fastEthernet0/24
no ip dhcp snooping trust
end
show ip dhcp snooping
```

All other ports on a fresh switch are untrusted by default. Confirm global snooping and VLAN 10 are active and that **no interface is trusted**. Do not apply the handout's nearby trust example yet.

**Screenshot now — `E2-10-no-trust-config.png`:** capture global/VLAN snooping state and interface trust information. If the output lists only explicitly configured interfaces, pair it with the relevant `show running-config` portion rather than inventing rows.

An existing lease can remain visible after enabling snooping, so it is not a valid acquisition test. Enter Simulation with a clean DHCP event list.

**PC1 → Desktop → Command Prompt:**

```text
ipconfig /release
ipconfig /renew
ipconfig /all
```

**Expected answer:** PC1 cannot complete a new legal lease because the server's Offer/ACK enters SW1 Fa0/1, still untrusted, and is discarded. PC1 may show failure, `0.0.0.0`, or an automatic `169.254.x.x` address depending on the build. None is a legal-pool lease. Capture an actual Offer at SW1 when supported; absence of a reply at PC1 alone does not identify the drop boundary. [1]

**Screenshot now — `E2-11-no-trust-failure.png`:** `-a` shows PC1's released/failed acquisition; `-b` shows the legal Offer reaching SW1 and being discarded if the simulator exposes it. Record an unavailable packet detail honestly.

**Step 6 evidence:** `E2-10-no-trust-config.png`, `E2-11-no-trust-failure.png`.

## Step 7 — Trust only the legal server port and reacquire a lease

**SW1 → CLI; start in EXEC mode:**

```text
enable
configure terminal
interface fastEthernet0/1
ip dhcp snooping trust
end
show ip dhcp snooping
```

Only **SW1 Fa0/1** must be trusted. Fa0/2, Fa0/23, and Fa0/24 remain untrusted. Trust is an **ingress** decision: legal replies enter Fa0/1 and may leave the untrusted client/trunk ports. SW1 Fa0/23 faces clients, not another server, so it does not need trust merely because it is a trunk.

**Screenshot now — `E2-12-legal-port-trusted.png`:** show Fa0/1 trusted, VLAN 10 active, and other relevant ports untrusted.

**PC1 → Desktop → Command Prompt:**

```text
ipconfig /release
ipconfig /renew
ipconfig /all
```

**Expected answers:** yes, PC1 should now acquire a legal-pool lease; yes, the legal server replies and its replies are admitted through Fa0/1. Inspect the actual Offer/ACK if the lease does not complete.

### Compatibility check: Option 82, only if needed

If legal DHCP worked before snooping but still fails after the correct trust configuration, first check VLANs, physical ports, the legal service, and pool availability. In Simulation, determine whether the request reaches the server and whether the server responds. Some Packet Tracer Server-PT implementations do not handle snooping's Option 82 as expected. For that specific compatibility case, disable insertion on **SW1**, keeping all trust decisions intact. [1]

**SW1 → CLI; optional compatibility correction, start in EXEC mode:**

```text
enable
configure terminal
no ip dhcp snooping information option
end
show ip dhcp snooping
```

Repeat PC1's release/renew block. Record that Option 82 insertion was disabled; do not call this a new trusted port or disable snooping to get a passing result. If the command is unsupported, record the rejection and the first failing packet boundary.

**SW2 remains a plain Layer-2 switch with snooping disabled throughout this primary experiment.** This avoids downstream Option 82 insertion and an upstream untrusted-port Option 82 conflict. Do not independently enable snooping on SW2 without a separately designed trust/Option 82 plan. A rogue on SW1 Fa0/24 is blocked before it can reach either switch's clients. A rogue attached to SW2 could, however, send directly to other SW2 clients without crossing SW1; those same-switch clients are **not locally protected** by this design. SW1's untrusted Fa0/23 protects the SW1 side from such replies, not SW2's entire broadcast domain.

**Screenshot now — `E2-13-legal-lease-restored.png`:** show PC1's renewed legal IP, `.1` gateway, and `10.0.0.10` DNS, plus the legal ACK as a second frame. If used, include the Option 82 disabled state as another frame.

**Step 7 evidence:** `E2-12-legal-port-trusted.png`, `E2-13-legal-lease-restored.png`.

## Step 8 — Reattach the rogue and prove its Offer is blocked

1. Keep legal DHCP On and SW1 Fa0/1 trusted.
2. Connect **ROGUE-DHCP FastEthernet0 → SW1 Fa0/24**, wait for forwarding, and confirm its DHCP service is On.
3. Enter Simulation, select DHCP, and clear old events.

**PC1 → Desktop → Command Prompt:**

```text
ipconfig /release
ipconfig /renew
ipconfig /all
```

4. Capture/Forward through the exchange. Open the rogue's outbound Offer and the corresponding event at SW1. It must arrive on **Fa0/24**. Inspect the processing/drop explanation and verify there is no forwarded rogue Offer toward PC1 or the trunk. Inspect the legal Offer/ACK separately.

**Answer:** DHCP snooping should discard the rogue's server messages received on untrusted Fa0/24, while permitting the legal replies entering trusted Fa0/1. A successful legal lease alone is not sufficient proof: the legal server could simply have won a race. The rogue Offer and its discard are the negative evidence. [1]

**Screenshot now — `E2-14-rogue-offer-dropped.png`:** frame `-a` shows a real Offer from `10.0.0.200`; frame `-b` shows its SW1 ingress/drop event; keep device/port names readable. If this Packet Tracer version does not model the discard details, show the available event path and explicitly mark the missing drop detail as a limitation.

**Screenshot now — `E2-15-protected-client.png`:** capture PC1's actual legal lease/DNS and the legal ACK from `10.0.0.10`. Do not reuse Step 7's screenshot.

Leave the rogue attached for the final protected topology.

**Step 8 evidence:** `E2-14-rogue-offer-dropped.png`, `E2-15-protected-client.png`.

## Step 9 — Display and explain SW1's binding table

Release and renew PC2 and PC3 after snooping is active so their leases are observed by SW1.

**PC2 → Desktop → Command Prompt:**

```text
ipconfig /release
ipconfig /renew
ipconfig /all
```

**PC3 → Desktop → Command Prompt:**

```text
ipconfig /release
ipconfig /renew
ipconfig /all
```

**SW1 → CLI; EXEC verification:**

```text
enable
show ip dhcp snooping binding
show ip dhcp snooping
```

**Screenshot now — `E2-16-binding-table.png`:** keep every binding-table column and total readable. Add frames of PC1/PC2/PC3 actual IP/MAC values if needed to match the rows.

Explain the actual rows, using these meanings rather than inventing sample leases:

| Column | Meaning and expected topology relationship |
|---|---|
| `MacAddress` | DHCP client's MAC address; compare with that PC's physical address. |
| `IpAddress` | Actual address learned from the accepted DHCP exchange. |
| `Lease(sec)` | Lease lifetime/remaining value displayed by this implementation; record it, do not assume a duration. |
| `Type` | Normally a dynamically learned `dhcp-snooping` entry. |
| `VLAN` | `10` in the switched experiment. |
| `Interface` | Client-facing ingress on **this switch**: PC1 on Fa0/2, PC2/PC3 reachable through Fa0/23. SW1 cannot show SW2's local Fa0/2/Fa0/3 as its own ports. |

A static server address is not automatically a dynamic binding. The rogue's blocked offers should not create valid client bindings. Bindings arise from DHCP exchanges observed after snooping is enabled, and releases/expiry can remove them. The database can be used by **Dynamic ARP Inspection** to validate ARP mappings and **IP Source Guard** to restrict source IP usage; enabling snooping alone does not automatically enable either feature. [1]

If the table is unexpectedly empty, verify that a complete legal ACK crossed SW1 after enabling snooping, VLAN 10 is active, and the client did not remain statically configured. If the simulator fails to populate a table despite an observed successful exchange, document that mismatch rather than typing fabricated rows.

**Step 9 evidence:** `E2-16-binding-table.png`.

## Step 10 — Limit PC1's port to 10 DHCP packets per second

**SW1 → CLI; start in EXEC mode:**

```text
enable
configure terminal
interface fastEthernet0/2
ip dhcp snooping limit rate 10
end
show ip dhcp snooping
show running-config
```

**Screenshot now — `E2-17-rate-limit.png`:** show Fa0/2 untrusted with rate 10 pps, or the Fa0/2 running configuration if the summary does not expose the rate. If the parser rejects the rate command, capture the literal error in this checkpoint and identify the unsupported model/build.

A DHCP starvation attempt sends many requests, often with different client identities, to exhaust the server's address pool. Rate limiting reduces the volume admitted from the user port; it is not a complete guarantee against every slow or distributed attack. **On supported Cisco IOS switches, exceeding the configured DHCP rate can put the interface into the error-disabled state**, interrupting traffic through that port until recovery. [1][4]

Do not use repeated ping, a simple ICMP PDU, or a few manual renew clicks as proof of a DHCP rate violation. Packet Tracer may expose the configuration but lack a usable DHCP flood generator or implement neither accurate pps enforcement nor the error-disable event. In that case, report **“10 pps configured; above-threshold behavior not empirically demonstrated in this Packet Tracer build”** and answer the IOS behavior theoretically with the source. If the rate command itself fails, report **“rate limit unsupported/not configured”** instead.

**SW1 → CLI; after ordinary renewal or a supported, isolated DHCP-only test:**

```text
enable
show interfaces fastEthernet0/2
show interfaces status
```

**Screenshot now — `E2-18-port-state.png`:** show the actual Fa0/2 status. If a real over-limit test is available, preserve its traffic-rate evidence and the actual error-disabled event as additional frames. Otherwise capture the normal port state and label the missing overload test accurately; never manufacture an err-disabled screenshot.

If you genuinely triggered error-disable, stop the offending traffic first, then recover the port manually.

**SW1 → CLI; recovery only after a real violation, start in EXEC mode:**

```text
enable
configure terminal
interface fastEthernet0/2
shutdown
no shutdown
end
show interfaces status
```

Renew PC1 once normally and verify legal acquisition. Save the final switched state:

**SW1 → CLI; EXEC mode:**

```text
enable
write memory
```

**SW2 → CLI; EXEC mode:**

```text
enable
write memory
```

Use **File → Save As → `E2-DHCP-SNOOPING.pkt`**. Final state: rogue attached to untrusted Fa0/24, only Fa0/1 trusted, VLAN 10 snooped on SW1, SW2 snooping disabled, Fa0/2 limited to 10 pps if supported, and the legal server On.

**Step 10 evidence:** `E2-17-rate-limit.png`, `E2-18-port-state.png`.

## Step 11 — Replace the switch role with a router and use ACLs

### 11.1 Operational interpretation: a routed relay, not Layer-2 snooping

The source asks to use a router instead of the switch and allow only the DHCP server to answer. An ordinary 2911 router does **not** bridge all its routed interfaces into the same VLAN. Assigning `10.0.0.0/24` to several routed interfaces creates overlapping connected subnets and is invalid. A router ACL also cannot inspect a rogue-to-client frame that stays on an external shared Layer-2 segment.

Therefore create a **new, separate** Packet Tracer file using a **2911 with three GigabitEthernet ports**, one client, and the two servers on **three distinct subnets**. Preserve client subnet `10.0.0.0/24`; move the servers to new subnets and configure DHCP relay. This changes the Figure 2 server addressing/topology only for Step 11. It is an explicit operational interpretation of the underspecified replacement requirement, **not a claim of literal same-LAN router replacement**. Retain Steps 1–10 unchanged in their `.pkt`.

Place `R-DHCP` (2911), `PC1-R` (PC-PT), `LEGAL-R` and `ROGUE-R` (Server-PT). Connect each endpoint directly to its router interface using the automatic cable selector or copper crossover. Do not add a shared client/rogue switch.

| Link | Router address/mask | Endpoint address/mask | Endpoint default gateway |
|---|---|---|---|
| R-DHCP G0/0 ↔ PC1-R FastEthernet0 | `10.0.0.1/24` | DHCP, legal `.100–.139` | DHCP `10.0.0.1` |
| R-DHCP G0/1 ↔ LEGAL-R FastEthernet0 | `10.0.10.1/24` | Static `10.0.10.10/24` | `10.0.10.1` |
| R-DHCP G0/2 ↔ ROGUE-R FastEthernet0 | `10.0.20.1/24` | Static `10.0.20.200/24` | `10.0.20.1` |

Here, unlike Steps 1–10, `.1` is an actual functioning client gateway. All three masks are `255.255.255.0`.

### 11.2 Configure the revised servers through the GUI

On **LEGAL-R → Desktop → IP Configuration → Static**, enter IP `10.0.10.10`, mask `255.255.255.0`, gateway `10.0.10.1`, and DNS `10.0.10.10`.

On **ROGUE-R → Desktop → IP Configuration → Static**, enter IP `10.0.20.200`, mask `255.255.255.0`, gateway `10.0.20.1`, and DNS `203.0.113.53`.

On each server's **Services → DHCP**, turn the service On and save the following pool. Remove/replace any old pool as in Step 1. These pools deliberately allocate from the **remote client subnet**, not the server NIC subnet: the relay's `giaddr` of `10.0.0.1` identifies the intended client scope. [2]

| DHCP GUI field | LEGAL-R | ROGUE-R (controlled negative test only) |
|---|---|---|
| Pool Name | `RELAY-LEGAL-USERS` | `RELAY-ROGUE-USERS` |
| Default Gateway | `10.0.0.1` | `10.0.0.254` (deliberately false) |
| DNS Server | `10.0.10.10` | `203.0.113.53` (deliberately false) |
| Start IP Address | `10.0.0.100` | `10.0.0.150` |
| Subnet Mask | `255.255.255.0` | `255.255.255.0` |
| Maximum Number of Users | `40` | `40` |
| TFTP Server, if shown | `0.0.0.0` | `0.0.0.0` |
| WLC Address, if shown | `0.0.0.0` | `0.0.0.0` |

**Screenshot now — `E2-19-router-topology-pools.png`:** frame `-a` shows the three directly attached subnets and interface labels; `-b` shows LEGAL-R's interface and saved pool; `-c` shows ROGUE-R's interface and saved pool. Use additional readable frames if necessary.

### 11.3 Configure routing, relay, and ingress ACLs

Policy: relay requests only to `10.0.10.10`; allow legal DHCP replies on the legal-server ingress; deny UDP source port 67 on the rogue and client ingresses. Other IP traffic is permitted so the DHCP test does not accidentally become a blanket connectivity block.

| Flow | Router boundary | Policy |
|---|---|---|
| Client UDP 68 → server UDP 67, including initial broadcast | G0/0 inbound | Permit client DHCP; relay initial broadcasts to legal helper |
| Legal server UDP 67 → relay `10.0.0.1` UDP 67 | G0/1 inbound | Permit exact server/relay tuple |
| Legal server UDP 67 → client subnet UDP 68 (direct renewal reply) | G0/1 inbound | Permit exact legal server to client subnet |
| Any other server-style UDP source port 67 on legal segment | G0/1 inbound | Deny |
| Any UDP source port 67 arriving from rogue segment | G0/2 inbound | Deny regardless of claimed source IP |
| Server-style UDP source port 67 arriving from client segment | G0/0 inbound | Deny; ordinary client source port 68 is not denied |
| Other IP traffic | All three inbound ACLs | Permit |

**R-DHCP → CLI; start in EXEC mode:**

```text
enable
configure terminal
hostname R-DHCP
service dhcp
ip access-list extended CLIENT-IN
deny udp any eq 67 any
permit udp any eq 68 any eq 67
permit ip any any
exit
ip access-list extended LEGAL-IN
permit udp host 10.0.10.10 eq 67 host 10.0.0.1 eq 67
permit udp host 10.0.10.10 eq 67 10.0.0.0 0.0.0.255 eq 68
deny udp any eq 67 any
permit ip any any
exit
ip access-list extended ROGUE-IN
deny udp any eq 67 any
permit ip any any
exit
interface gigabitEthernet0/0
ip address 10.0.0.1 255.255.255.0
ip helper-address 10.0.10.10
ip access-group CLIENT-IN in
no shutdown
exit
interface gigabitEthernet0/1
ip address 10.0.10.1 255.255.255.0
ip access-group LEGAL-IN in
no shutdown
exit
interface gigabitEthernet0/2
ip address 10.0.20.1 255.255.255.0
ip access-group ROGUE-IN in
no shutdown
end
write memory
show ip interface brief
show ip route
show access-lists
show ip interface gigabitEthernet0/0
show ip interface gigabitEthernet0/1
show ip interface gigabitEthernet0/2
```

`service dhcp` enables the relay facility; **no router DHCP pool is configured**. Put `ip helper-address` on the **client-facing G0/0**, never on the server-facing port. The three connected `/24` routes appear automatically when the interfaces are up/up, so **no static route or default route is needed**. The server's own gateway must be `10.0.10.1`, providing the return path to relay `10.0.0.1`. The pool's client gateway is separately `10.0.0.1`. [2]

**Screenshot now — `E2-20-router-relay-acls.png`:** `-a` shows up/up interfaces and the three connected routes; `-b` shows ACL contents; `-c` and further frames show the helper on G0/0 and the three inbound ACL attachments. ACL existence alone does not prove it is applied.

### 11.4 Positive test: acquire a legal relayed lease

On **PC1-R → Desktop → IP Configuration**, select **DHCP**. Then enter Simulation mode, filter DHCP, and start a clean acquisition:

**PC1-R → Desktop → Command Prompt:**

```text
ipconfig /release
ipconfig /renew
ipconfig /all
```

Follow the request from the client to G0/0, the relayed request toward LEGAL-R, the legal reply to the router, and the final reply to PC1-R. Inspect `giaddr = 10.0.0.1` if displayed. Expect a legal `.100–.139` client lease, gateway `10.0.0.1`, and DNS `10.0.10.10`. Do not assign a static client IP to hide a relay failure.

**PC1-R → Desktop → Command Prompt, after a real lease is obtained:**

```text
ping 10.0.0.1
ping 10.0.10.10
```

A first ARP-related timeout can occur; repeat once after ARP settles. These pings test routed reachability, not DHCP authorization by themselves.

**R-DHCP → CLI; EXEC verification after the exchange:**

```text
enable
show access-lists
```

**Screenshot now — `E2-21-router-legal-dhcp.png`:** capture PC1-R's legal IP settings (`-a`), the real relayed request/legal reply (`-b`), and the actual LEGAL-IN permit counters and ping results (`-c` or further frames). If counters are not exposed by the simulator, keep packet-path evidence and state that limitation.

### 11.5 Negative test: prove the rogue ingress ACL, not just lack of broadcast forwarding

With only a legal helper, ROGUE-R never receives the client's broadcast. “The rogue did not answer” in that state proves isolation/relay selection, **not an ACL hit**. To exercise the ACL using a real DHCP response in this isolated `.pkt`, temporarily add the rogue as a second helper while leaving every ACL active.

**R-DHCP → CLI; temporary negative-test setup, start in EXEC mode:**

```text
enable
configure terminal
interface gigabitEthernet0/0
ip helper-address 10.0.20.200
end
show access-lists
```

Record the current ROGUE-IN deny count. Clear the Simulation event list and release/renew PC1-R again.

**PC1-R → Desktop → Command Prompt:**

```text
ipconfig /release
ipconfig /renew
ipconfig /all
```

Capture/Forward until the relayed request reaches ROGUE-R and its Offer returns to router **G0/2**. Its UDP source port 67 matches ROGUE-IN's first deny regardless of whether it targets relay port 67 or client port 68. The legal path remains permitted. Compare counters before and after; do not state an invented packet count.

**R-DHCP → CLI; EXEC verification:**

```text
enable
show access-lists
```

**Screenshot now — `E2-22-router-rogue-denied.png`:** `-a` shows the actual rogue Offer reaching G0/2 and being dropped; `-b` shows ROGUE-IN's real deny-counter change if implemented; `-c` shows PC1-R still using legal settings. If this Server-PT build cannot serve a relayed remote scope, stop at the actual failing boundary and report that negative DHCP-response test as unverified. A generic UDP test would establish only the ACL predicate, not a completed rogue DHCP exchange; never label it as a real Offer.

**Mandatory rollback immediately after the negative test:** remove the temporary rogue helper. Do not leave it in the saved design.

**R-DHCP → CLI; start in EXEC mode:**

```text
enable
configure terminal
interface gigabitEthernet0/0
no ip helper-address 10.0.20.200
end
write memory
show ip interface gigabitEthernet0/0
show access-lists
```

Repeat one ordinary PC1-R release/renew. Verify the only helper is `10.0.10.10`, all ingress ACLs remain attached, and the client still acquires a legal lease.

**Screenshot now — `E2-23-router-final-state.png`:** show the sole legal helper and active ACLs (`-a`), then PC1-R's final legal configuration (`-b`). Save via **File → Save As → `E2-ROUTER-ACL.pkt`**.

**Security boundary answer:** the client-ingress ACL stops server-style DHCP traffic that actually enters the router from the client segment. It cannot stop a rogue and a victim on the same external switch exchanging Layer-2 frames without crossing the router. This guide's direct client-to-router link excludes that bypass. If multiple clients share a switch, add switch DHCP snooping/client isolation rather than claiming the router ACL alone protects their local DHCP exchanges. The legal-server ACL trusts a specific IP **on a dedicated ingress segment**, not cryptographic server identity; it is not general authentication against a compromised legal server or arbitrary port-changing attacks.

**Step 11 evidence:** `E2-19-router-topology-pools.png`, `E2-20-router-relay-acls.png`, `E2-21-router-legal-dhcp.png`, `E2-22-router-rogue-denied.png`, `E2-23-router-final-state.png`.

## Complete evidence and submission checklist

There are **23 base screenshot checkpoints**. Optional letter-suffixed readability frames do not increase this base count. These are capture requirements, not a claim that screenshots already exist. If a simulator feature is unavailable, keep the corresponding error/configuration evidence and explicitly identify which expected observation could not be demonstrated.

### Topology and baseline: Steps 1–5

- [ ] `E2-01-topology.png` — Figure 2 topology, interface labels, rogue initially disconnected.
- [ ] `E2-02-vlan-trunks.png` — both switches' VLAN/access/trunk state.
- [ ] `E2-03-legal-server.png` — legal static NIC and saved pool.
- [ ] `E2-04-rogue-server.png` — rogue static NIC and saved pool.
- [ ] `E2-05-current-ip.png` — PC1 before renewal.
- [ ] `E2-06-renew.png` — actual renewal and configuration.
- [ ] `E2-07-legal-dora.png` — actual legal server identification.
- [ ] `E2-08-rogue-baseline.png` — rogue acquisition with the legal-service state stated.
- [ ] `E2-09-baseline-restored.png` — legal service restored, rogue disconnected, legal lease.

### Snooping and rate limit: Steps 6–10

- [ ] `E2-10-no-trust-config.png` — snooping global/VLAN state, no trusted port.
- [ ] `E2-11-no-trust-failure.png` — released lease followed by failed acquisition/drop.
- [ ] `E2-12-legal-port-trusted.png` — only SW1 Fa0/1 trusted.
- [ ] `E2-13-legal-lease-restored.png` — legal acquisition after trust; Option 82 adjustment if used.
- [ ] `E2-14-rogue-offer-dropped.png` — real rogue Offer and SW1 discard boundary.
- [ ] `E2-15-protected-client.png` — fresh legal result with rogue attached.
- [ ] `E2-16-binding-table.png` — actual binding rows, all columns, client correlation.
- [ ] `E2-17-rate-limit.png` — 10 pps configuration or exact unsupported-command evidence.
- [ ] `E2-18-port-state.png` — real interface state, with overload test limitation if applicable.

### Routed alternative: Step 11

- [ ] `E2-19-router-topology-pools.png` — separate subnets and revised server scopes.
- [ ] `E2-20-router-relay-acls.png` — connected routes, helper, ACL contents and attachments.
- [ ] `E2-21-router-legal-dhcp.png` — successful legal relay exchange and connectivity, if observed.
- [ ] `E2-22-router-rogue-denied.png` — actual rogue reply denied, or exact unverified boundary.
- [ ] `E2-23-router-final-state.png` — rogue test helper removed, final legal acquisition.

### Final delivery

- [ ] Answer every question using your actual observations: current IP; renewal; selected server; rogue timing/availability; no-trust failure; trusted legal success; blocked rogue Offer; binding-column meanings; 10 pps and real IOS error-disable behavior; router ACL/relay behavior and local Layer-2 limitation.
- [ ] State any unsupported commands, missing packet details, unavailable flood test, or unverified Step 11 relay/negative test without inventing output.
- [ ] State the Experiment 1-only address transformation scope and the separate routed interpretation used in Step 11.
- [ ] Include `E2-DHCP-SNOOPING.pkt`, `E2-ROUTER-ACL.pkt`, the report, actual screenshots, and any other experiment files required by the assignment. Include `E2-BASELINE.pkt` if created and useful.
- [ ] Save, close, and reopen both final `.pkt` files. Confirm their intended topology/configuration persisted; rerun an ordinary legal acquisition in each before final submission.
- [ ] Include sources used to answer explanatory questions, as required on page 8 of the assignment. Do not present reference-platform behavior as measured Packet Tracer behavior.
- [ ] Put **all experiments' required files**, including Experiment 1's files prepared separately, into **`FullName-6-2-8.zip`**. This guide alone is not a complete lab submission.
- [ ] Open the ZIP and check it contains the intended files, then extract it into a temporary folder and open both `.pkt` files from the extracted copy. Do not package missing images as if they existed.

## References for explanatory answers

The following official Cisco sources were located during authoring; [1] was read directly. They ground the protocol/IOS explanations, **not a claim that the full command set is implemented in Packet Tracer**.

1. **Cisco — Operate and Troubleshoot DHCP Snooping on Catalyst 9000 Switches.** Trusted/untrusted ingress behavior, global/VLAN configuration, Option 82 control, rate-limit syntax, binding-table columns, DAI/IP Source Guard relationship. This is an IOS XE reference; use the 2960 source and your actual PT build for platform behavior. https://www.cisco.com/c/en/us/support/docs/ip/dynamic-host-configuration-protocol-dhcp-dhcpv6/217055-operate-and-troubleshoot-dhcp-snooping.html
2. **Cisco — Configuring the Cisco IOS DHCP Relay Agent, IP Addressing: DHCP Configuration Guide, IOS 15M&T.** `service dhcp`, client-facing `ip helper-address`, relay forwarding, `giaddr`, and return-path concepts. https://cisco.com/c/en/us/td/docs/ios-xml/ios/ipaddr_dhcp/configuration/15-mt/dhcp-15-mt-book/config-dhcp-relay-agent.pdf
3. **Cisco — Catalyst 2960 DHCP configuration guide, Release 15.2(4)E.** Model-specific DHCP snooping, DHCP rate limiting, and error-disabled behavior reference; this official URL was verified in the parent review. https://cisco.com/c/en/us/td/docs/switches/lan/catalyst2960/software/release/15-2_4_e/configurationguide/b_1524e_consolidated_2960p_2960c_cg/m_1522e_sec_dhcp_2960_cg.html

4. **Cisco — Recover Errdisable Port State on Cisco IOS Platforms.** Explicitly lists DHCP snooping rate-limit as an error-disable cause and explains the resulting loss of forwarding. https://www.cisco.com/c/en/us/support/docs/lan-switching/spanning-tree-protocol/69980-errdisable-recovery.html

Primary assignment mapping: page 6 Figure 2 → Step 1; page 7 numbered items 1–8 → Steps 1–8; page 8 continuation and items 9–11 → Steps 8–11; page 8 submission/reference notes → this final checklist. The full provided text was read, including the Experiment 1-only transformation instruction and the source's inconsistent header/example device labels.
