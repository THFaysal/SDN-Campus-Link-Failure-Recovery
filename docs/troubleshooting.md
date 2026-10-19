# Troubleshooting the GNS3 / FAUCET lab

| Symptom | What to check |
| --- | --- |
| A GNS3 Docker node won't start | Is the correct container image available on the configured GNS3 compute? The `.gns3` file was exported from a specific compute and references `:latest` images. |
| FAUCET cannot see switches | OVS `eth0` and controller are on the same management segment, the controller listens on TCP 6653, and `ovs-vsctl list controller` reports an active connection. |
| `br0` not found | Confirm the installed OVS Docker image's bridge naming/initialization. The report used `br0`; it is not guaranteed by a different image version. |
| No flows installed | Inspect FAUCET logs, validate YAML syntax and DPIDs/port numbering, and confirm OpenFlow 1.3 + secure fail mode. |
| Hosts can't ping server | Verify host addresses and VLAN 100 access ports, link states, controller connectivity, and correct endpoint cabling. |
| Unexpected loop or silent switch fallback | Inter-switch links must be configured as FAUCET stack ports; do not treat stack ports as ordinary VLAN access ports. Check secure fail mode. |
| Backup route fails after OVS2-OVS3 is disabled | Confirm OVS1-OVS4 and OVS4-OVS3 links are up and both ends are defined in the stack configuration. Check port counters on OVS4. |
| `iperf3` unavailable | Install or select a compatible image containing `iperf3` in all test clients and server. |
| The diagram loads but the experiment does not | The `.gns3` file includes topology metadata only. The original controller YAML is in `configs/faucet.yaml`, but host IP assignments and persistent switch settings must still be restored separately. |

To avoid mistakes, verify Linux interface names and OpenFlow ports explicitly. The report used a data-interface mapping where GNS3/Linux `eth1` often appeared as OpenFlow port 2; do not assume the same mapping in every Docker version.
