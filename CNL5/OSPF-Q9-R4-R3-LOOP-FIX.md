# OSPF Question 9 — R4 and R3 routing-loop fix

The alternating hops `10.10.16.2` and `10.10.10.1` mean R4 is forwarding the
PC1 packet to R3 while R3 is forwarding it back to R4. The old Experiment 4
static routes are overriding OSPF:

- R4 has a static PC1 route through R3.
- R3 has a static default route through R4.

Power R2 back on before applying the fix.

## Run on R4

```text
enable
configure terminal
no ip route 10.10.9.0 255.255.255.0 10.10.10.1
no ip route 10.10.8.0 255.255.255.0 10.10.10.1
no ip route 10.10.6.0 255.255.255.0 10.10.10.1
no ip route 0.0.0.0 0.0.0.0 10.10.12.2
interface FastEthernet1/0
ip ospf cost 1000
no shutdown
exit
router ospf 1
no passive-interface FastEthernet0/1
no passive-interface FastEthernet1/0
end
```

## Run on R3

```text
enable
configure terminal
no ip route 10.10.16.0 255.255.255.0 10.10.10.2
no ip route 10.10.8.0 255.255.255.0 10.10.9.1
no ip route 10.10.6.0 255.255.255.0 10.10.9.1
no ip route 0.0.0.0 0.0.0.0 10.10.10.2
router ospf 1
no passive-interface FastEthernet0/0
no passive-interface FastEthernet0/1
end
```

## Verify before turning R2 off again

Run on R4:

```text
show ip ospf neighbor
show ip route 10.10.6.0
show ip route 0.0.0.0
```

R5 router ID `5.5.5.5` must be `FULL`. With R2 on, the PC1 route and default
route must be OSPF routes through R3 at `10.10.10.1`; neither route may begin
with `S`.

Run on R3:

```text
show ip route 10.10.6.0
show ip route 0.0.0.0
```

Both routes must be OSPF routes through R2 at `10.10.9.1`; neither route may
begin with `S`.

## Repeat the R2 failure

Start on PC3:

```text
ping -n 1000 10.10.6.1
```

After five replies, power off the complete R2 from its `Physical` tab. Wait a
few seconds, then run on R4:

```text
show ip route 10.10.6.0
show ip route 0.0.0.0
```

Both routes must now use R5 at `10.10.12.2`. Only then run on PC3:

```text
tracert 10.10.6.1
```

The recovered path must be:

```text
10.10.16.2
10.10.12.2
10.10.11.1
10.10.6.1
```

It must not alternate between `10.10.16.2` and `10.10.10.1`.
