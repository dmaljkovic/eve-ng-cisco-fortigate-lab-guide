# Lab overview and design

## Goals

Build two reproducible labs with a small, fixed image set:

1. **SOHO lab** — one FortiGate firewall, one Cisco L2 switch, two Linux PCs.
2. **Small-enterprise lab** — one FortiGate edge firewall, one Cisco core
   router, two Cisco access switches, four Linux PCs, and two VLANs.

Both labs use a Cisco router as a stand-in for the Internet. This keeps the lab
self-contained and repeatable. If you want real Internet access, replace the ISP
router with an EVE-NG Cloud connected to a NAT-enabled host interface; see
[docs/07-node-settings.md](07-node-settings.md).

## Design choices

- **FortiGate at the edge.** The FortiGate provides WAN connectivity, NAT,
  firewall policy, and DHCP (SOHO) or default routing (enterprise).
- **Cisco for routing and switching.** Dynamips routers cover IOS routing;
  IOL L2 images cover Cisco switching. The c3725 is the default core router
  because it is small, fast enough, and supports optional switch modules.
- **Static routing by default.** It is easier to troubleshoot than a routing
  protocol. OSPF is an optional extension in the enterprise lab.
- **Two VLANs in the enterprise lab.** VLAN 10 (users) and VLAN 20 (servers or
  guest). The core router performs inter-VLAN routing with 802.1Q subinterfaces.
- **Closed-lab ISP.** A c3725 with a loopback of `8.8.8.8/32` proves that NAT and
  default routing work without depending on the host network.
- **Small resource footprint.** The 16 GiB EVE-NG VM can run either lab
  comfortably when Slax PCs are limited to 1 GiB each.

## Addressing plan — SOHO lab

| Segment | Network | Device address | Notes |
|---|---|---|---|
| ISP ↔ FGT WAN | `203.0.113.0/30` | ISP `203.0.113.1`, FGT `203.0.113.2` | Point-to-point link |
| ISP loopback | `8.8.8.8/32` | ISP `8.8.8.8` | Stand-in for an Internet host |
| FGT LAN | `192.168.10.0/24` | FGT `192.168.10.1` | DHCP pool `.100`–`.199` |
| PCs | `192.168.10.0/24` | DHCP `.100`–`.199` | Slax clients |

## Addressing plan — small-enterprise lab

| Segment | Network | Device address | Notes |
|---|---|---|---|
| ISP ↔ FGT WAN | `203.0.113.0/30` | ISP `203.0.113.1`, FGT `203.0.113.2` | Point-to-point link |
| ISP loopback | `8.8.8.8/32` | ISP `8.8.8.8` | Stand-in for an Internet host |
| FGT ↔ CORE | `10.0.0.0/30` | FGT `10.0.0.1`, CORE `10.0.0.2` | Routed transit link |
| VLAN 10 | `10.10.0.0/24` | CORE `10.10.0.1` | Users; DHCP pool `.100`–`.199` |
| VLAN 20 | `10.20.0.0/24` | CORE `10.20.0.1` | Servers/guest; DHCP pool `.100`–`.199` |
| PCs | VLAN 10 or 20 | DHCP | Address depends on the access port VLAN |

## Resource budget

Recommended node RAM:

| Node | RAM | CPU |
|---|---|---|
| FortiGate 7.0.9 | 2 GB | 1 |
| IOL L2 switch | 1 GB | 1 |
| IOL L3 router | 1 GB | 1 |
| c1710 | 96 MB | 1 |
| c3725 | 256 MB | 1 |
| c7200 | 512 MB | 1 |
| Slax Linux PC | 1 GB | 1 |

Approximate totals:

- **SOHO:** FGT 2 GB + IOL L2 1 GB + two Slax PCs 2 GB + ISP c3725 256 MB ≈ **5.3 GB**
- **Enterprise:** FGT 2 GB + c3725 256 MB + two IOL L2 2 GB + four Slax PCs 4 GB
  + ISP c3725 256 MB ≈ **8.5 GB**

Both fit in a 16 GiB EVE-NG VM. Avoid starting every node with default RAM;
the Slax template defaults to 4 GB and the FortiGate template defaults to 1 GB.

## What is installed

The current image set is deliberately small:

- FortiGate `7.0.9`
- Slax Linux `9.11.0`
- Cisco Dynamips: c1710, c3725, c7200 (15.2(4)M11)
- Cisco IOL: two L2 switch images, two L3 router images, plus `iourc`

Removed as unnecessary for these labs: FortiAnalyzer (FAZ), FortiManager (FMG),
pfSense, the NSE5 topology labs, the c3640, and duplicate c7200/IOL variants.

## What to read next

- [02-eve-ng-vm.md](02-eve-ng-vm.md) — reference EVE-NG VM and image paths
- [03-image-inventory.md](03-image-inventory.md) — exact image files and templates
- [04-soho-lab.md](04-soho-lab.md) — build the SOHO lab
- [05-small-enterprise-lab.md](05-small-enterprise-lab.md) — build the enterprise lab
