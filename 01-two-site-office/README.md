# Lab 01 — Two-Site Office with an Isolated IoT VLAN

> **Status:** 🚧 In progress. Build phases are tracked at the bottom of this page.

## Goal

Design and build a small two-site company network (HQ + one branch) with separate Staff, Guest, IoT, and Management networks. IoT sensors are isolated so they can reach **only** their data server, similar to the distributed sensor networks I work with in research. The branch reaches the internet through HQ.

## Topology

```
                         [ ISP ] ── [ Internet-Web 198.51.100.10 ]
                            │ 203.0.113.0/30
                         [HQ-R1]  ── 10.255.0.0/30 ──  [BR-R1]
                            │ 10.255.1.0/30               │ trunk (router-on-a-stick)
                        [HQ-MLS1]                      [BR-SW1]
                       ║        │                     │   │   │
            EtherChannel        │                  Staff Guest IoT
            (LACP, 2 links)     │
                    [HQ-SW1]  [HQ-SW2]
                    │  │  │      │
               Staff Guest IoT  Servers
```

*(A draw.io diagram will replace this sketch as `topology.png`.)*

## Devices

| Device | Model (Packet Tracer) | Role |
|---|---|---|
| ISP | ISR 4331 | Simulated internet provider |
| HQ-R1 | ISR 4331 | Edge router: NAT/PAT, OSPF, default route |
| HQ-MLS1 | Catalyst 3650 | HQ core: SVIs for inter-VLAN routing, OSPF |
| HQ-SW1, HQ-SW2 | Catalyst 2960 | HQ access switches |
| BR-R1 | ISR 4331 | Branch router: router-on-a-stick, OSPF |
| BR-SW1 | Catalyst 2960 | Branch access switch |
| SRV-CORE | Server | DHCP (all pools) + DNS |
| SRV-LOG | Server | Syslog + NTP |
| SRV-IOT | Server | IoT data server |

HQ and the branch use two different inter-VLAN routing methods on purpose (Layer 3 switch SVIs vs. router-on-a-stick).

## VLAN plan

| VLAN | Name | Purpose |
|---|---|---|
| 10 | STAFF | Employee PCs |
| 20 | GUEST | Visitor devices, internet only |
| 30 | IOT | Sensors, data server access only |
| 40 | SERVERS | HQ servers |
| 99 | MGMT | Switch/router management (SSH) |
| 666 | BLACKHOLE | Unused ports (shut down) |
| 999 | NATIVE | Trunk native VLAN, carries no user traffic |

## IP addressing

**Convention:** `.1` = gateway, `.2–.99` = static/reserved, `.100–.199` = DHCP pool.

| Site | VLAN | Subnet | Gateway | Gateway lives on |
|---|---|---|---|---|
| HQ | 10 Staff | 10.10.10.0/24 | 10.10.10.1 | HQ-MLS1 SVI |
| HQ | 20 Guest | 10.10.20.0/24 | 10.10.20.1 | HQ-MLS1 SVI |
| HQ | 30 IoT | 10.10.30.0/24 | 10.10.30.1 | HQ-MLS1 SVI |
| HQ | 40 Servers | 10.10.40.0/24 | 10.10.40.1 | HQ-MLS1 SVI |
| HQ | 99 Mgmt | 10.10.99.0/24 | 10.10.99.1 | HQ-MLS1 SVI |
| Branch | 10 Staff | 10.20.10.0/24 | 10.20.10.1 | BR-R1 G0/0/1.10 |
| Branch | 20 Guest | 10.20.20.0/24 | 10.20.20.1 | BR-R1 G0/0/1.20 |
| Branch | 30 IoT | 10.20.30.0/24 | 10.20.30.1 | BR-R1 G0/0/1.30 |
| Branch | 99 Mgmt | 10.20.99.0/24 | 10.20.99.1 | BR-R1 G0/0/1.99 |

**Point-to-point links and loopbacks**

| Link | Subnet | Addresses |
|---|---|---|
| HQ-R1 ↔ ISP | 203.0.113.0/30 | ISP .1, HQ-R1 .2 |
| HQ-R1 ↔ BR-R1 | 10.255.0.0/30 | HQ-R1 .1, BR-R1 .2 |
| HQ-R1 ↔ HQ-MLS1 | 10.255.1.0/30 | HQ-R1 .1, HQ-MLS1 .2 (routed port) |
| Loopbacks (OSPF router ID) | /32 | HQ-R1 1.1.1.1, HQ-MLS1 3.3.3.3, BR-R1 2.2.2.2 |

**Static hosts**

| Host | Address |
|---|---|
| SRV-CORE (DHCP/DNS) | 10.10.40.10 |
| SRV-LOG (Syslog/NTP) | 10.10.40.11 |
| SRV-IOT (IoT data) | 10.10.40.20 |
| HQ-SW1 / HQ-SW2 mgmt | 10.10.99.11 / 10.10.99.12 |
| BR-SW1 mgmt | 10.20.99.11 |
| Internet-Web | 198.51.100.10 |

## Security policy (ACLs)

| Source | Allowed | Blocked |
|---|---|---|
| IoT (both sites) | DHCP; SRV-IOT only | Everything else, logged |
| Guest (both sites) | DHCP; DNS on SRV-CORE; internet | All other 10.0.0.0/8 addresses |
| Staff | Everything | — |
| SSH to network devices | Mgmt VLANs only (10.10.99.0/24, 10.20.99.0/24) | All other sources; Telnet disabled |

## Design decisions

*To be written during the build. For each: what I chose, what I considered, and why.*

- Why OSPF instead of static routes between sites
- Why ACLs are applied inbound on the IoT and Guest gateways
- Why DHCP is centralized with `ip helper-address` instead of a pool on each router
- Why the branch uses router-on-a-stick while HQ uses a Layer 3 switch

## Build phases

- [ ] **1. Topology + VLANs:** devices cabled, VLANs created, access ports assigned, unused ports shut into VLAN 666
- [ ] **2. Trunks + EtherChannel:** 802.1Q trunks with native VLAN 999, LACP bundle HQ-MLS1 ↔ HQ-SW1
- [ ] **3. Inter-VLAN routing:** SVIs at HQ, router-on-a-stick at the branch
- [ ] **4. OSPF:** single area 0, loopback router IDs, passive user-facing interfaces
- [ ] **5. Services:** central DHCP with relay, DNS, NTP, Syslog
- [ ] **6. Edge:** default route + `default-information originate`, NAT/PAT overload on HQ-R1
- [ ] **7. Security:** IoT/Guest ACLs, SSH-only management, port security on access ports
- [ ] **8. Verification + troubleshooting log**
- [ ] *Stretch:* IPv6 on the Staff VLAN, HSRP gateway redundancy

## Verification

Outputs are saved in [`verification/`](verification/). See that folder's README for the command checklist.

## Troubleshooting

See [`troubleshooting.md`](troubleshooting.md).

## Lessons learned

*To be written after the build.*
