# FAUCET controller configuration (original lab file)

**[`faucet.yaml`](faucet.yaml) is the original configuration copied from the FAUCET container (`/etc/faucet/faucet.yaml`) and supplied by the project author.** It replaces the earlier reconstructed example.

It defines:

- `users` VLAN 100 for H1-Video, H2-Business, H3-File, and Server.
- Four Open vSwitch datapaths (`OVS1` through `OVS4`) with recorded controller-facing datapath IDs.
- OVS1 as the FAUCET stacking root (`stack.priority: 1`).
- Four bidirectional stacking links: OVS1-OVS2, OVS2-OVS3, OVS3-OVS4, and OVS4-OVS1.
- Two potential paths between the client-side OVS1 and server-side OVS3.

## Validation performed on the published text

The saved YAML was parsed with PyYAML. The VLAN, four unique datapath IDs, four access ports, and all four reciprocal stack-link definitions were checked for consistency. **This is a structural check, not a successful runtime FAUCET validation.**

## Run this config in your own lab

1. Restore the GNS3 topology and start FAUCET and the OVS nodes.
2. Place this file at `/etc/faucet/faucet.yaml` in the FAUCET container, after backing up any existing file.
3. On each OVS container, compare the actual OpenFlow port mappings with the YAML (`ovs-ofctl -O OpenFlow13 show br0`).
4. Compare bridge datapath IDs with the YAML and check the controller's connection to each datapath.
5. Run the FAUCET-version-appropriate `check_faucet_config` validation or read startup logs before testing.

**Remaining dependencies:** This configuration does not contain IP interface assignments, Docker/GNS3 runtime storage, OVS bridge bootstrapping, or traffic-test logs. Those must be restored separately; do not claim that this repository alone reproduces the full experiment without them.
