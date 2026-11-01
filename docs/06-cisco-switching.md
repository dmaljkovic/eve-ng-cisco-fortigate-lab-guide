# Cisco switching options

Short answer: **yes, you have Cisco switches** — the two IOL L2 images. The
Dynamips c1710, c3725, and c7200 are routers, although the c3725 can take a
16-port switch module.

## Summary

| Option | What it is | Switchports? | Recommended use |
|---|---|---|---|
| IOL L2 | Cisco IOS-on-Linux L2 switch image | Yes, configurable count | Primary Cisco switch for VLAN/trunk/STP labs |
| c3725 + `NM-16ESW` | Dynamips router with a 16-port switch module | Yes, 16 ports per module | Router+switch combo, older IOS practice |
| c3725 alone | Dynamips router | No | Routing only |
| c7200 | Dynamips router with PA Ethernet adapters | No | Core/PE routing only |
| c1710 | Dynamips small router | No | Small branch routing |

There is **no Catalyst, IOSvL2, or vIOS-L2 image** installed. For simple
VLAN/trunk/STP/EtherChannel labs, IOL L2 is the right tool. You can add a
dedicated IOSvL2/vIOS-L2 image later if you need a different platform.

## IOL L2 — recommended Cisco switch

Installed images:

- `L2-ADVENTERPRISEK9-M-15.2-20150703.bin` — advanced enterprise; feature-rich.
- `i86bi_linux_l2-ipbasek9-ms.high_iron_aug9_2017b.bin` — IP Base; lighter.

EVE-NG template: `Cisco IOL` (node name `R`).

Settings:

- Select the L2 `.bin` in the Image dropdown.
- Set **Ethernet** to 4 or 8 (the template default is 1).
- RAM: 1 GB.
- Console: telnet.

Example access switch:

```text
hostname ACCESS1

vlan 10
 name USERS
vlan 20
 name SERVERS
exit

interface Ethernet0/0
 description TRUNK-TO-CORE
 switchport mode trunk
 switchport trunk allowed vlan 10,20
! If your image supports it:
! switchport trunk encapsulation dot1q

interface Ethernet0/1
 description PC-VLAN10
 switchport mode access
 switchport access vlan 10

interface Ethernet0/2
 description PC-VLAN20
 switchport mode access
 switchport access vlan 20
```

Notes:

- The `switchport trunk encapsulation dot1q` command is accepted by some IOL
  images and rejected by others. If it is rejected, omit it; the trunk uses
  802.1Q by default on these images.
- Use the L2 image for switchports. An L3 image is a router and does not provide
  `switchport` commands.
- IOL L2 supports the usual VLAN, trunk, STP, and EtherChannel CLI. It is not a
  Catalyst hardware emulation, so a few hardware-specific commands may differ.

Useful verification:

```text
show vlan brief
show interfaces trunk
show interfaces status
show interfaces switchport
show spanning-tree
```

## c3725 + NM-16ESW — router with a switch module

The c3725 template exposes two network-module slots. Supported modules:

- `NM-1FE-TX` — one FastEthernet routed port.
- `NM-16ESW` — sixteen 10/100 switchports.

Add `NM-16ESW` in slot 1 or slot 2 in the node settings, then stop/wipe/start the
node. The switchports appear as `Ethernet1/0`–`Ethernet1/15` (slot 1) or
`Ethernet2/0`–`Ethernet2/15` (slot 2). The router's onboard ports remain
`FastEthernet0/0` and `FastEthernet0/1`.

IOS 12.4T on this platform uses VLAN database mode:

```text
vlan database
 vlan 10 name USERS
 vlan 20 name SERVERS
 exit

interface Ethernet1/0
 description PC-VLAN10
 switchport mode access
 switchport access vlan 10

interface Ethernet1/1
 description PC-VLAN20
 switchport mode access
 switchport access vlan 20

interface Ethernet1/15
 description TRUNK-TO-CORE
 switchport mode trunk
 switchport trunk allowed vlan 10,20
```

Verify:

```text
show vlan
show interfaces switchport
show interfaces trunk
```

Limitations:

- 10/100 switchports only.
- Older IOS 12.4T feature set compared with the IOL L2 images.
- The switch module lives inside a Dynamips router; it is not a standalone
  Catalyst.
- Useful when you want one node to be both a router and a small switch, or when
  practicing IOS 12.4T switch commands.

## c7200 — router only

The c7200 template supports routed port adapters:

- `PA-FE-TX` — one FastEthernet port.
- `PA-4E` — four Ethernet ports.
- `PA-8E` — eight Ethernet ports.

These are routed interfaces, not switchports. The c7200 template has no
`NM-16ESW` option, so you cannot run `switchport` commands on it. Use it for
core/PE routing, BGP, MPLS, and many routed interfaces; use IOL L2 for switching.

## c1710 — router only

The c1710 is a small branch router. It has built-in Ethernet ports but is not a
switch platform. Use it for small-site routing, NAT, ACLs, and DHCP. Use IOL L2
for access switching.

## Which should I use?

- **Learning VLANs, trunks, STP, EtherChannel, port security:** IOL L2.
- **One node that is both a router and a small switch:** c3725 + `NM-16ESW`.
- **Core/PE routing, BGP, MPLS, many routed ports:** c7200.
- **Small branch router:** c1710.
- **Dedicated modern virtual L2 switch platform:** add an IOSvL2/vIOS-L2 image
  if you have one; not required for this guide.

## Extensions

- Add a third IOL L2 switch and another switch-to-switch trunk.
- Configure Rapid PVST+ if the image supports it:
  ```text
  spanning-tree mode rapid-pvst
  ```
- Add an EtherChannel between two switches:
  ```text
  interface range Ethernet0/2 - 3
   channel-group 1 mode active
  ```
- Add an SVI for management:
  ```text
  interface Vlan10
   ip address 10.10.0.2 255.255.255.0
   no shutdown
  ```
  Keep inter-VLAN routing on the core router in these labs; use the SVI only for
  management or as an extension.
