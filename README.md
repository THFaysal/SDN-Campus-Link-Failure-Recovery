# SDN-Based Automatic Traffic Rerouting & Link Failure Recovery

**A redundant campus-network lab using GNS3, FAUCET, Open vSwitch, and OpenFlow 1.3.**

This Advanced Networking Lab project examines what happens when an inter-switch link fails in a single-path network, and how an SDN-controlled redundant topology restores connectivity through an alternate path. The experiment uses a centrally managed Layer 2 switching fabric with three traffic services: UDP video-style traffic, TCP business traffic, and TCP file-transfer traffic.

![GNS3 SDN campus network topology](assets/topology.png)

## Highlights

- **10 virtual devices:** 1 FAUCET controller, 1 management switch, 4 Open vSwitch datapaths, 3 client hosts, and 1 server.
- **Control plane:** FAUCET over OpenFlow 1.3 (TCP 6653) on `10.0.0.0/24`.
- **Data plane:** VLAN 100, `192.168.10.0/24`, using FAUCET stacking.
- **Controller configuration:** Original [`configs/faucet.yaml`](configs/faucet.yaml) recovered from the FAUCET container; four datapaths and four reciprocal stack links.
- **Primary path:** H1/H2/H3 -> OVS1 -> OVS2 -> OVS3 -> Server.
- **Alternate path:** H1/H2/H3 -> OVS1 -> OVS4 -> OVS3 -> Server.
- **Failure tested:** The OVS2-to-OVS3 link is deliberately disabled.
- **Measured with:** `ping`, `iperf3`, and Open vSwitch counters.

## Selected results

| Observation | Recorded outcome |
| --- | --- |
| Single-path baseline after link failure | 44.8718% ICMP loss in a captured run; **no automatic recovery** |
| Redundant SDN after link failure | 21.843% ICMP loss in a separate captured run; **automatic recovery** |
| Repeat SDN ping evidence run | Approx. **13.2 s** network connectivity interruption |
| UDP video-style flow (6 Mbit/s offered) | **4.41 Mbit/s** received, **27%** datagram loss, **1.602 ms** jitter |
| TCP business flow | **103** retransmissions, approx. **26 s** zero-throughput interval |
| TCP file flow | **177** retransmissions, approx. **27 s** zero-throughput interval |
| Alternate-path verification | Increased OVS4 port counters while the primary link was down |

**Interpretation:** The baseline and redundant-network ping runs have different durations, and the baseline was restored manually. Their loss percentages are observations from those runs, **not** a controlled percentage-improvement or guaranteed convergence-time claim. See [Results & limitations](docs/results.md).

## Repository contents

```text
.
|-- README.md
|-- .gitignore
|-- assets/
|   `-- topology.png
|-- gns3/
|   `-- Sdn_Project.gns3
|-- configs/
|   |-- README.md
|   `-- faucet.yaml
`-- docs/
    |-- architecture.md
    |-- setup-and-config.md
    |-- experiments.md
    |-- results.md
    |-- troubleshooting.md
    |-- github-publishing-checklist.md
    `-- SDN_Lab_Report_Public.pdf
```

## Get started

1. Install GNS3 and a compatible Docker/GNS3 compute environment.
2. Open [`gns3/Sdn_Project.gns3`](gns3/Sdn_Project.gns3).
3. Confirm the required container images, interface numbering, and bridge names in the imported nodes.
4. Set controller/switch management addresses and host data-plane addresses.
5. Restore the included [original FAUCET YAML](configs/faucet.yaml), configure OpenFlow 1.3, and validate the actual interface mapping and controller connectivity.
6. Run baseline and redundant failure scenarios using the documented commands.

**Reproducibility note:** This repository now includes the [original FAUCET controller configuration](configs/faucet.yaml), copied from `/etc/faucet/faucet.yaml` in the project's controller container and supplied by the author. The YAML parses and its stack links are reciprocal, but it has **not been executed or checked with a running FAUCET instance in this package**. The `.gns3` file still contains topology metadata, not complete saved Linux IP assignments, OVS bridge startup scripts, Docker container state, or raw `iperf3` logs. Full replay requires restoring these runtime settings.

Detailed instructions: [Setup](docs/setup-and-config.md) | [Experiments](docs/experiments.md) | [Troubleshooting](docs/troubleshooting.md).

## Documentation

- [Network architecture, devices, ports, and IP addressing](docs/architecture.md)
- [Environment and configuration instructions](docs/setup-and-config.md)
- [Original FAUCET configuration and validation notes](configs/README.md)
- [Failure-injection and test procedure](docs/experiments.md)
- [Measured results, interpretation, limitations](docs/results.md)
- [Troubleshooting guide](docs/troubleshooting.md)
- [GitHub publication checklist](docs/github-publishing-checklist.md)
- [Public academic lab report (student ID redacted)](docs/SDN_Lab_Report_Public.pdf)

## Scope and limitations

This is a **virtual GNS3 lab** focused on same-subnet Layer 2 SDN resilience, not a deployed enterprise network. BGP, MPLS, inter-subnet routing, and QoS scheduling were outside this project's experimental scope. Recovery was **not instantaneous**. Measurements depend on the emulated environment and were not repeated enough to support broad statistical claims.

## Project information

- **Author:** Tanvirul Hoque Faysal
- **Institution:** Southeast University, Department of Computer Science and Engineering
- **Course:** CSE348.1, Advanced Networking LAB
- **Lab report submitted:** September 2026
- **Technologies:** GNS3, Docker-based nodes, FAUCET, Open vSwitch, OpenFlow 1.3, Linux, ICMP, TCP/UDP, `iperf3`.

The public report hides the student's university ID. No open-source license is assigned in this package; choose one intentionally before inviting reuse or contributions.
