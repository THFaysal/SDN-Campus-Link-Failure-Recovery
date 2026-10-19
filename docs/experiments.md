# Experiment procedure and commands

The experiment compares the impact of the **same physical failure** (OVS2-to-OVS3) under two conditions: (A) alternate path disabled and (B) FAUCET-controlled redundant stacking enabled.

**Run these commands only in an isolated, disposable GNS3 lab.** Interface names below are taken from the report and must be confirmed in your local topology. Make sure you can restore the links.

## Preconditions

- Management interfaces can reach FAUCET (`10.0.0.10:6653`).
- All four OVS controllers report connected/active.
- The host VLAN is 100 and `192.168.10.11` can ping `192.168.10.100`.
- Both redundant data-plane paths are initially available.

```sh
# Example on an OVS instance
ovs-vsctl list controller
ovs-ofctl -O OpenFlow13 dump-flows br0
```

## A. Baseline: remove backup path

```sh
# On OVS1: disable OVS1 -> OVS4 backup link
ip link set eth5 down

# On H1: begin continuous pings
ping -i 0.2 192.168.10.100

# On OVS2: fail the only remaining complete path
ip link set eth2 down
```

Observe loss of reachability. This baseline does not reroute automatically; recovery requires an operator to restore the failed link. Stop the ping and record its summary.

**Restore both links** before the next experiment:

```sh
# On OVS2
ip link set eth2 up

# On OVS1
ip link set eth5 up
```

Verify ping works again and the controller has updated flows/stacking state.

## B. Redundant SDN: allow automatic rerouting

Keep OVS1 `eth5` **up** so the alternate path through OVS4 exists.

```sh
# On H1: begin continuous pings
ping -i 0.2 192.168.10.100

# On OVS2: fail OVS2 -> OVS3 primary link
ip link set eth2 down
```

Watch for replies resuming **without manually reconfiguring a forwarding path**. Use timestamps or missed ping sequence numbers to estimate the interruption, and inspect OVS4 counters to confirm traffic uses the alternate route.

```sh
# On OVS4: the report identifies OF port 3 as the link toward OVS3
ovs-ofctl -O OpenFlow13 dump-ports br0 3
```

Observe counter changes while the failed OVS2-to-OVS3 link remains unavailable.

## C. Three concurrent traffic services

Start separate `iperf3` listeners on the server:

```sh
iperf3 -s -p 5201 -D
iperf3 -s -p 5202 -D
iperf3 -s -p 5203 -D
```

Run clients from separate host consoles (simultaneously if possible):

```sh
# H1: UDP video-style traffic, 6 Mbit/s
iperf3 -c 192.168.10.100 -p 5201 -u -b 6M -t 60 -i 1

# H2: TCP business traffic, 3 Mbit/s
iperf3 -c 192.168.10.100 -p 5202 -b 3M -t 60 -i 1

# H3: TCP file-transfer traffic, 8 Mbit/s
iperf3 -c 192.168.10.100 -p 5203 -b 8M -t 60 -i 1
```

During the 60 s runs, inject the same `OVS2 eth2 down` event. Record receiver throughput, UDP loss/jitter, TCP retransmission counts, zero-throughput windows, and when traffic returns.

## D. Clean up and document

```sh
# On OVS2 - restore primary transit link
ip link set eth2 up

# On OVS1 - make sure alternate path is restored
ip link set eth5 up
```

Suggested evidence artifacts to add later: screenshots of `ovs-vsctl list controller`, `ovs-ofctl ... dump-flows`, time-stamped ping output, raw `iperf3` outputs, and paired OVS4 port counter captures. The current package contains the author's report and its evidence figures, not raw command logs.

## Measuring and interpreting outages

- ICMP estimate used in the report: missing ping sequences x `0.2 s`.
- Baseline restoration depends on manual intervention: **do not present it as a measured SDN convergence time**.
- The UDP datagram loss and TCP retransmission/zero-throughput interval measurements describe **different behavior and cannot be reduced to one latency improvement percentage**.
- See [Results](results.md) for the actual reported observations and limitations.
