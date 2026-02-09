# BGP EVPN VXLAN Fabric on Cisco Catalyst 9000

This repository documents the configuration steps for deploying a BGP EVPN VXLAN fabric using Cisco Catalyst 9000 series switches running IOS-XE 17.x.

## Topology Overview

![Network Topology](diagrams/topology.svg)

### ASCII Diagram

```
                    +------------------------+
                    |   External_Router_1    |
                    |      (AS 6505)         |
                    |   E0/0         E0/1    |
                    +----+-------------+-----+
                         |             |
                    192.168.220.x  192.168.221.x
                         |             |
                    +----+-------------+-----+
                    |         sw1            |
                    |     (Spine/RR)         |
                    |    Loopback: 1.1.1.1   |
                    |      (AS 6500)         |
                    +----+-----------+-------+
                         |           |
                    Gi1/0/1      Gi1/0/2
                         |           |
              +----------+           +----------+
              |                                 |
         Gi1/0/1                           Gi1/0/1
    +----+--------+                   +----+--------+
    |     sw2     |                   |     sw3     |
    | (Leaf VTEP) |                   | (Leaf VTEP) |
    | Lo0: 1.1.1.2|                   | Lo0: 1.1.1.3|
    +------+------+                   +------+------+
           |                                 |
      VLAN 1151                         VLAN 1151
      VLAN 1152                         VLAN 1152
      VLAN 1153                         VLAN 1153
           |                                 |
    +------+------+                   +------+------+
    |  Desktop 0  |                   |  Desktop 1  |
    | 192.168.1.x |                   | 192.168.1.x |
    +-------------+                   +-------------+
```

## Network Details

| Component | Value |
|-----------|-------|
| BGP AS (Fabric) | 6500 |
| BGP AS (External) | 6505 |
| Underlay Protocol | OSPF 100 |
| Overlay Protocol | BGP EVPN |
| VRF | VRF-1 (RD: 10:1101) |

## VLANs and VNI Mapping

| VLAN | Name | VNI | Subnet | Purpose |
|------|------|-----|--------|---------|
| 1101 | VRF1_CORE_VLAN | 50101 | - | L3VNI (Core) |
| 1151 | VRF1_ACCESS_VLAN | 401151 | 192.168.1.0/25 | Access VLAN 1 |
| 1152 | VRF1_ACCESS_VLAN_2 | 401152 | 192.168.2.0/25 | Access VLAN 2 |
| 1153 | VRF_ACCESS_VLAN_3 | 401153 | 192.168.3.0/25 | Access VLAN 3 |

## Device Roles

| Device | Role | Loopback IP | Management IP |
|--------|------|-------------|---------------|
| sw1 | Spine / Route Reflector | 1.1.1.1 | 192.168.20.77 |
| sw2 | Leaf VTEP | 1.1.1.2 | 192.168.20.76 |
| sw3 | Leaf VTEP | 1.1.1.3 | 192.168.20.81 |
| External_Router_1 | External Router | 10.1.1.1 | 192.168.20.165 |

## Configuration Steps

Follow these guides in order:

1. [Step 1: Base Configuration](docs/01-base-config.md) - Hostname, Loopbacks, IP routing
2. [Step 2: Underlay (OSPF)](docs/02-underlay-ospf.md) - OSPF configuration for reachability
3. [Step 3: BGP EVPN Overlay](docs/03-bgp-evpn-overlay.md) - BGP configuration for EVPN
4. [Step 4: VRF Configuration](docs/04-vrf-config.md) - VRF definition with route targets
5. [Step 5: VLAN and VNI Mapping](docs/05-vlan-vni-mapping.md) - VLAN to VNI association
6. [Step 6: L2VPN EVPN Instance](docs/06-l2vpn-evpn-instance.md) - EVPN instance configuration
7. [Step 7: NVE Interface](docs/07-nve-interface.md) - VXLAN tunnel endpoint
8. [Step 8: SVI Configuration](docs/08-svi-config.md) - VLAN interfaces for routing
9. [Step 9: Verification Commands](docs/09-verification.md) - Show commands to verify setup
10. [Step 10: Live Verification Output](docs/10-verification-output.md) - Actual output from running fabric

## Quick Start - Adding a New VLAN

To extend a new VLAN (e.g., VLAN 1154) across the VXLAN fabric:

```
! 1. Create VLAN
vlan 1154
 name NEW_ACCESS_VLAN

! 2. Create L2VPN EVPN Instance
l2vpn evpn instance 1154 vlan-based
 encapsulation vxlan
 route-target export 1154:1
 route-target import 1154:1

! 3. Map VLAN to VNI
vlan configuration 1154
 member evpn-instance 1154 vni 401154

! 4. Add VNI to NVE Interface
interface nve1
 member vni 401154 ingress-replication local-routing

! 5. Create SVI (Optional - for inter-VLAN routing)
interface Vlan1154
 vrf forwarding VRF-1
 ip address 192.168.4.2 255.255.255.128
```

## Device Configurations

Complete running configurations are available in the [configs/](configs/) directory:

- [sw1 (Spine)](configs/sw1.cfg)
- [sw2 (Leaf)](configs/sw2.cfg)
- [sw3 (Leaf)](configs/sw3.cfg)
- [External_Router_1](configs/external_router_1.cfg)

## Requirements

- Cisco Catalyst 9000 Series switches
- IOS-XE 17.x or later
- License: Network Advantage + DNA Advantage

## References

- [Cisco EVPN VXLAN Configuration Guide](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9000/software/release/17-x/configuration_guide/vxlan/b_17x_vxlan_9k_cg.html)
- [Cisco BGP EVPN Configuration Guide](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/nexus9000/sw/7-x/vxlan/configuration/guide/b_Cisco_Nexus_9000_Series_NX-OS_VXLAN_Configuration_Guide_7x/b_Cisco_Nexus_9000_Series_NX-OS_VXLAN_Configuration_Guide_7x_chapter_0100.html)
