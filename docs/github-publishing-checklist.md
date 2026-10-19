# GitHub publication checklist

## Files already prepared

- [x] `README.md` with topology, project scope, technology stack, results, and limitations.
- [x] `gns3/Sdn_Project.gns3` original topology definition.
- [x] `assets/topology.png` original topology illustration.
- [x] `docs/SDN_Lab_Report_Public.pdf` report with student ID redacted on cover page.
- [x] Architecture, setup, testing, results, and troubleshooting documentation.
- [x] Original FAUCET `faucet.yaml`, recovered from `/etc/faucet/faucet.yaml` in the controller container and supplied by the author.
- [x] `.gitignore` covering typical secrets, captures, and runtime files.

## Before claiming complete reproducibility

- [x] Recover the **actual** FAUCET configuration used in the lab and review the shared text for secrets (none apparent).
- [ ] Export any OVS bridge startup commands and host IP configuration scripts used during the experiment.
- [ ] Add sanitized terminal evidence/raw `ping` and `iperf3` logs if retained.
- [ ] Record pinned container image versions and GNS3/FAUCET compatibility.
- [x] Check the YAML syntax and reciprocal stack-link mappings.
- [ ] Validate the original YAML inside a running FAUCET container, using your installed controller version.
- [ ] Optionally add a short GIF/video showing the link-failure recovery (only from your real lab, not generated or simulated).
- [ ] Decide whether to add an open-source `LICENSE` (this package does not choose one for you).

## Before making the repository public

- [ ] Check screenshots, reports, configs, and logs for passwords, secrets, university/student IDs, local IPs you do not want public, and any personal metadata.
- [ ] Verify the report is permitted to be published under your institution's course rules.
- [ ] Confirm names/attribution if contributions were made by other people.
- [ ] Read the README and confirm each result is backed by your original report.
- [x] Describe the YAML accurately as the author-supplied original controller configuration; distinguish static parsing from runtime validation.

## Suggested GitHub repository name

`sdn-campus-link-failure-recovery`

Suggested description:

> GNS3 lab demonstrating FAUCET/OpenFlow 1.3 traffic rerouting after an OVS link failure, with baseline comparison and ICMP/iperf3 measurements.

Suggested GitHub topics:

`sdn`, `gns3`, `faucet`, `openvswitch`, `openflow13`, `networking`, `network-resilience`, `link-failure`, `iperf3`, `docker`

## Upload using the GitHub website

1. Create a new repository (preferably with no auto-generated README).
2. Upload the **contents** of this folder, not just the ZIP file.
3. Commit using a message such as `Document GNS3 SDN failover lab and experimental results`.
4. Check that the topology image renders in README and internal links open.

## Upload using Git

From inside the folder containing `README.md`:

```sh
git init
git add .
git commit -m "Add SDN campus network lab and documentation"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/sdn-campus-link-failure-recovery.git
git push -u origin main
```

Replace `YOUR_USERNAME` and create the empty remote repo first. Uploading the ZIP **as one file** will not display the README at the repository root.
