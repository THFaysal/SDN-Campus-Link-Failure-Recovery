# Setup and configuration

These instructions describe how to **restore** the reported lab. The author subsequently recovered and supplied the original FAUCET YAML from the controller container. The `.gns3` metadata still does not include complete saved Linux runtime or Docker state.

## Prerequisites

- GNS3 with a compatible Docker-capable local or VM compute server.
- Access to container images referenced in the GNS3 project: `faucet/faucet`, `gns3/openvswitch`, and `gns3/alpinet`.
- Linux networking commands, Open vSwitch tools (`ovs-vsctl`, `ovs-ofctl`), ICMP `ping`, and `iperf3` in their respective containers.
- A configuration file accessible to the FAUCET container (for example, a bind-mounted `/etc/faucet/faucet.yaml`, depending on image and platform).

## 1. Open the topology

Open `gns3/Sdn_Project.gns3` in GNS3, verify all 10 nodes and 13 links, and review Docker image/template settings. The `.gns3` file references container IDs and a VM compute from the original environment; local Docker/GNS3 templates may need reconfiguration.

**Data-plane cabling from the project JSON:**

- OVS1 `eth1` -> H1; `eth2` -> H2; `eth3` -> H3; `eth4` -> OVS2; `eth5` -> OVS4.
- OVS2 `eth1` -> OVS1; `eth2` -> OVS3.
- OVS3 `eth1` -> OVS2; `eth2` -> OVS4; `eth3` -> Server.
- OVS4 `eth1` -> OVS1; `eth2` -> OVS3.
- Controller and each OVS `eth0` connect to MGMT-SW.

The actual OpenFlow interface numbers must be verified after the OVS bridge initializes.

## 2. Assign management and host addresses

Use the lab's reported management subnet `10.0.0.0/24` on controller/OVS **management `eth0`** interfaces:

```text
FAUCET  10.0.0.10/24
OVS1    10.0.0.11/24
OVS2    10.0.0.12/24
OVS3    10.0.0.13/24
OVS4    10.0.0.14/24
```

For example, on the OVS1 management console, **only if not already assigned**:

```sh
ip addr add 10.0.0.11/24 dev eth0
ip link set eth0 up
```

On the user/server containers, configure `eth0` in VLAN 100's access segment:

```text
H1      192.168.10.11/24
H2      192.168.10.12/24
H3      192.168.10.13/24
Server  192.168.10.100/24
```

For example, on H1, **only if not already assigned**:

```sh
ip addr add 192.168.10.11/24 dev eth0
ip link set eth0 up
```

There is no default gateway requirement for same-subnet tests. Existing addresses may need removal first.

## 3. Configure FAUCET and OpenFlow

The included [`configs/faucet.yaml`](../configs/faucet.yaml) is the **original FAUCET configuration** supplied by the author from `/etc/faucet/faucet.yaml` in the project's running controller container. The earlier reconstructed example has been removed to avoid ambiguity.

Before applying it:

1. Check bridge names (`br0`) and verify the actual `ovs-ofctl -O OpenFlow13 show br0` port map on all switches.
2. Check that reported datapath IDs match `ovs-ofctl` and OVS state in your recreated environment.
3. Back up any existing controller configuration and place the supplied `configs/faucet.yaml` at `/etc/faucet/faucet.yaml` in the controller container. Validate it with `check_faucet_config` appropriate to your FAUCET version or inspect controller startup logs.
4. Check controller reachability over the management network.

On **each** OVS instance, the reported controller settings are:

```sh
ovs-vsctl set bridge br0 protocols=OpenFlow13
ovs-vsctl set-controller br0 tcp:10.0.0.10:6653
ovs-vsctl set-fail-mode br0 secure
ovs-vsctl list controller
ovs-ofctl -O OpenFlow13 dump-flows br0
```

The recorded experiment verified `is_connected=true` and `state=ACTIVE` for all four controller sessions before performing host traffic tests. If this check fails, don't proceed to failure experiments.

## 4. Verify baseline connectivity

With all redundant physical links initially up:

```sh
# In H1
ping -c 5 192.168.10.100

# In any OVS container
ovs-ofctl -O OpenFlow13 dump-flows br0
```

Run `iperf3` servers on the Server node as described in [Experiments](experiments.md).

## 5. Validate the topology against your installation

Different GNS3/Docker image builds may use different interface ordering, default bridges, FAUCET config paths, or OpenFlow versions. The recovered controller YAML is provided separately, but the `.gns3` file still does not embed the controller bind mount or persist OVS startup commands and host IP assignments. Save and publish those **actual** files later for complete reproducibility.

Additional reading: [Architecture](architecture.md) | [Troubleshooting](troubleshooting.md).
