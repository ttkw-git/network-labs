# Verification Checklist — Lab 01

Save each output as a `.txt` file named `<device>_<command>.txt` (for example `HQ-MLS1_show-ip-ospf-neighbor.txt`).

| Feature | Command | Run on | Expected result |
|---|---|---|---|
| VLANs | `show vlan brief` | All switches | Ports in correct VLANs; unused ports in 666 |
| Trunks | `show interfaces trunk` | All switches | Native VLAN 999; allowed VLANs listed |
| EtherChannel | `show etherchannel summary` | HQ-MLS1, HQ-SW1 | Port-channel flagged SU, members P |
| Inter-VLAN | `show ip interface brief` | HQ-MLS1, BR-R1 | SVIs / subinterfaces up/up |
| OSPF | `show ip ospf neighbor` | HQ-R1, HQ-MLS1, BR-R1 | All neighbors FULL |
| Routing | `show ip route` | BR-R1 | O routes for HQ subnets; O*E2 default route |
| DHCP | `show ip dhcp binding` or host `ipconfig` | Hosts / SRV-CORE | Hosts get .100+ addresses at both sites |
| NAT | `show ip nat translations` | HQ-R1 | Translations after pinging Internet-Web |
| ACLs | `show access-lists` | HQ-MLS1, BR-R1 | Match counters increasing |
| SSH | `show ip ssh`, `show running-config \| section line vty` | All devices | SSH v2 on; `transport input ssh` |
| Port security | `show port-security interface <int>` | Access switches | Secure-up, sticky MAC learned |

**Policy tests (host-to-host)**

| Test | Expected |
|---|---|
| IoT host → SRV-IOT (10.10.40.20) | ✅ Success |
| IoT host → Staff PC | ❌ Blocked |
| Guest host → Internet-Web (198.51.100.10) | ✅ Success |
| Guest host → Staff PC or servers | ❌ Blocked |
| Branch Staff PC → Internet-Web | ✅ Success (through HQ NAT) |
| SSH from Staff PC to a switch | ❌ Refused |
| SSH from Mgmt VLAN to a switch | ✅ Success |
