# Network Labs — Tony Wong

**CCNA · CompTIA A+** · Winnipeg, MB · [Portfolio](https://ttkw-git.github.io/) · [LinkedIn](https://www.linkedin.com/in/tony-tk-wong/)

Hands-on Cisco labs, documented the way a real IT team would hand them over: topology, addressing plan, device configs, verification output, and a log of faults I broke on purpose and then diagnosed.

| # | Lab | Skills | Tool | Status |
|---|-----|--------|------|--------|
| 01 | [Two-Site Office with an Isolated IoT VLAN](01-two-site-office/) | VLANs, trunking, EtherChannel, inter-VLAN routing, OSPF, DHCP relay, ACLs, NAT/PAT, SSH hardening | Cisco Packet Tracer | 🚧 In progress |
| 02 | Config Backup & Health Check with Python | Python, Netmiko, SSH automation | Cisco Modeling Labs | 📋 Planned |

## How each lab is organized

```
NN-lab-name/
├── README.md            case study: goal, topology, addressing, design decisions, results
├── topology.png         network diagram
├── *.pkt                Packet Tracer file
├── configs/             running-config for each device
├── verification/        show-command output proving each feature works
└── troubleshooting.md   faults introduced on purpose: symptom → diagnosis → fix → verification
```
