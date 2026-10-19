# Experimental results and limitations

This page summarizes the **recorded measurements in the supplied academic report**. These are historical lab observations; they were not rerun or independently benchmarked when creating this repository.

## Connectivity and failover

| Test | Ping samples | Loss | Recovery behavior |
| --- | --- | --- | --- |
| Normal, initial five-packet reference | 5 transmitted, 5 received | 0% | Working; mean RTT 19.246 ms |
| Normal, longer redundant stable run | 133 transmitted, 133 received | 0% | Working; mean RTT 11.652 ms |
| Single-path baseline, OVS2-OVS3 failed | 390 transmitted, 215 received | 44.8718% | No automatic recovery; manually restored |
| Redundant SDN, OVS2-OVS3 failed | 293 transmitted, 229 received | 21.843% | Automatic restoration through alternate path |
| Separate repeat SDN evidence capture | Replies observed through seq. 18; resumed at seq. 85 | 66 missed 0.2 s probes | Approx. 13.2 s ICMP connectivity interruption |

**Do not infer a controlled 51% improvement from the two packet-loss percentages.** The baseline and redundant runs had different total durations, and restoration in the baseline happened manually. The repeat 13.2 s ICMP interruption comes from a different evidence capture than the 21.843% full run.

## Three-service test (60 s)

| Client / service | Offered rate | Receiver / average result | Failure evidence |
| --- | --- | --- | --- |
| H1 video (UDP) | 6.00 Mbit/s | 4.41 Mbit/s received; 31.5 MB; 1.602 ms jitter | 8248/31078 datagrams lost (27%) |
| H2 business (TCP) | 3.00 Mbit/s | 3.01 Mbit/s average; 21.5 MB | 103 retransmissions; ~26 s zero-throughput interval |
| H3 file (TCP) | 8.00 Mbit/s | 8.00 Mbit/s average; 57.2 MB | 177 retransmissions; ~27 s zero-throughput interval |

The TCP averages include post-recovery retransmission/catch-up traffic. Their average throughput must **not** be interpreted as uninterrupted service. UDP datagrams transmitted during the outage were not recovered by the transport protocol.

## Alternate-path confirmation

When the primary link was unavailable, reported OVS4 port 3 counters increased between measurements:

| Counter | First reading | Second reading | Increase |
| --- | ---: | ---: | ---: |
| RX packets | 515,590 | 528,567 | 12,977 |
| TX packets | 599,887 | 655,766 | 55,879 |
| RX bytes | 43,241,261 | 44,151,108 | 909,847 |
| TX bytes | 199,329,789 | 284,152,680 | 84,822,891 |

This was used as switch-level evidence that OVS4 handled traffic while the OVS2-OVS3 link was down.

## Limits of the evidence

1. This was a **GNS3/Docker virtual lab**, not a live hardware campus network.
2. Virtualization and host scheduling influence observed performance and convergence times.
3. Tests were not identical in duration; baseline restoration was manual.
4. The three `iperf3` tests were started manually and not perfectly synchronized.
5. Only the OVS2-to-OVS3 failure scenario was evaluated.
6. No evidence in the report demonstrates controller redundancy, production deployment, inter-subnet routing, BGP, MPLS, QoS scheduling, or general-purpose congestion-aware load balancing.

## Next improvements (not yet implemented here)

- The **actual** controller YAML is now saved in `configs/faucet.yaml`; publish repeatable OVS/node configuration and startup scripts when available.
- Store raw, time-stamped test outputs for independent verification.
- Run automated, synchronized failure injection with repeated trials.
- Pin Docker images by digest and document GNS3 + FAUCET version compatibility.
- Test additional failures and possible controller redundancy.

**Primary source:** [SDN Lab Report - Public PDF](SDN_Lab_Report_Public.pdf), sections 7-10.
