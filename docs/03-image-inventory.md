# Image inventory

The labs use a deliberately small set of images. This page explains what each
file is, where it comes from, which EVE-NG template to use, and how it is used.

> Vendor images are **not** stored in this repository. The files below are
> installed on the EVE-NG VM. Use only images you are licensed to use.

## Quick reference

| File or folder | Type | EVE template | Role |
|---|---|---|---|
| `fortinet-FGT-v7.0.9-build0444/` | QEMU qcow2 | `Fortinet` | Edge firewall |
| `linux-slax-9.11.0/` | QEMU qcow2 | `Linux` | Client PC |
| `c1710-bk9no3r2sy-mz.124-23.image` | Dynamips IOS | `1710` | Small branch router |
| `c3725-adventerprisek9-mz.124-15.T14.image` | Dynamips IOS | `3725` | Main lab / core router |
| `c7200-adventerprisek9-mz.152-4.M11.image` | Dynamips IOS | `7206VXR` | Core / PE router |
| `L2-ADVENTERPRISEK9-M-15.2-20150703.bin` | IOL L2 | `Cisco IOL` | Cisco L2 switch |
| `i86bi_linux_l2-ipbasek9-ms.high_iron_aug9_2017b.bin` | IOL L2 | `Cisco IOL` | Cisco L2 switch, IP Base |
| `L3-ADVENTERPRISEK9-M-15.2-M5.3.bin` | IOL L3 | `Cisco IOL` | Lightweight router |
| `i86bi_LinuxL3-AdvEnterpriseK9-M2_157_3_May_2018.bin` | IOL L3 | `Cisco IOL` | Lightweight router, newer |
| `iourc` | IOL license | — | Required for IOL nodes |

## QEMU images

### FortiGate 7.0.9

- **Folder:** `/opt/unetlab/addons/qemu/fortinet-FGT-v7.0.9-build0444/`
- **Disk:** `virtioa.qcow2` (about 74 MB)
- **EVE template:** `Fortinet` (description: "Fortinet FortiGate")
- **Template defaults:** 1 CPU, 1 GB RAM, 4 NICs, virtio NIC, telnet console
- **Recommended:** 2 GB RAM
- **Source:** Fortinet FortiGate VM KVM image (`fortinet-FGT-v7.0.9-build0444`).
  Proprietary; evaluation or licensed.
- **Role:** Edge firewall, router, NAT, DHCP, VPN, UTM.
- **How to use:** Add a `Fortinet` node, select
  `fortinet-FGT-v7.0.9-build0444`, connect `port1` to the WAN side and `port2`
  to the LAN side. Log in on the telnet console as `admin`, set a strong
  password, then apply the configs in `configs/fortigate/`.
- **Notes:** FortiOS 7.0.9 is from 2022. It is fine for these labs. FortiGate VM
  evaluation has throughput and feature limitations; some features require a
  license.

### Slax Linux 9.11.0

- **Folder:** `/opt/unetlab/addons/qemu/linux-slax-9.11.0/`
- **Disk:** `hda.qcow2` (about 1.3 GB)
- **EVE template:** `Linux`
- **Template defaults:** 2 CPU, 4 GB RAM, 1 NIC, virtio NIC, VNC console,
  `-boot order=cd -cpu host`
- **Recommended:** 1 CPU, 1 GB RAM for a PC
- **Source:** Slax, a lightweight Linux distribution, packaged as an EVE-NG
  qcow2. Free/open source.
- **Role:** Client PC or lightweight server.
- **How to use:** Add a `Linux` node, select `linux-slax-9.11.0`, lower RAM to
  1 GB, connect the NIC to a switch access port, and use the VNC console. Run
  `dhclient eth0` (or `udhcpc -i eth0`) to get a DHCP address. See
  `configs/linux/slax-client.md`.

## Dynamips router images

Dynamips runs real Cisco IOS router images. The `.image` files live in
`/opt/unetlab/addons/dynamips/`. They are proprietary Cisco IOS images supplied
by you; Dynamips itself is open source.

### Cisco 1710

- **File:** `c1710-bk9no3r2sy-mz.124-23.image`
- **IOS:** 12.4(23)
- **EVE template:** `1710`
- **Template defaults:** 96 MB RAM, 128 KB NVRAM, telnet console, idlepc
  `0x80369ac4`
- **Role:** Small branch/SOHO router.
- **How to use:** Small branch routing, NAT, ACLs, DHCP. Use `show ip interface
  brief` to confirm interface names. Good for a remote branch; the c3725 is more
  flexible for the main lab.

### Cisco 3725

- **File:** `c3725-adventerprisek9-mz.124-15.T14.image`
- **IOS:** 12.4(15)T14
- **EVE template:** `3725`
- **Template defaults:** 256 MB RAM, 128 KB NVRAM, telnet console, idlepc
  `0x60c08728`
- **Slots:** two network-module slots. Supported modules:
  - `NM-1FE-TX` — one FastEthernet port
  - `NM-16ESW` — sixteen 10/100 switch ports
- **Role:** Main lab router; optional core router.
- **How to use:** OSPF/EIGRP, NAT, ACLs, DHCP, inter-VLAN routing, WAN links.
  Add `NM-16ESW` if you want a router with a built-in switch module. See
  [06-cisco-switching.md](06-cisco-switching.md).

### Cisco 7200 (7206VXR)

- **File:** `c7200-adventerprisek9-mz.152-4.M11.image`
- **IOS:** 15.2(4)M11
- **EVE template:** `7206VXR`
- **Template defaults:** 256 MB RAM, 128 KB NVRAM, NPE-400, telnet console,
  idlepc `0x62f21000`
- **Slots:** six port-adapter slots. Supported modules:
  - `PA-FE-TX` — one FastEthernet port
  - `PA-4E` — four Ethernet ports
  - `PA-8E` — eight Ethernet ports
- **Recommended:** 512 MB RAM or more for IOS 15.2
- **Role:** Core/PE router, many routed interfaces.
- **How to use:** Larger topologies, BGP, MPLS, traffic engineering. It is a
  router; the template has no switch module. Overkill for SOHO.

## IOL images

Cisco IOL ("IOS on Linux") runs natively on Linux; it does not use QEMU. The
`.bin` files live in `/opt/unetlab/addons/iol/bin/` and must be executable.
IOL nodes also need the `iourc` license file.

Use the `Cisco IOL` template (the node name is `R`). The template defaults to
one Ethernet interface; increase the interface count in the node settings.

### IOL L2 — Cisco switches

These are the Cisco switches for the labs.

- **`L2-ADVENTERPRISEK9-M-15.2-20150703.bin`**
  - 15.2-era L2 advanced-enterprise image.
  - Feature-rich L2 switch image; recommended for VLAN, trunk, STP, and
    EtherChannel labs.
- **`i86bi_linux_l2-ipbasek9-ms.high_iron_aug9_2017b.bin`**
  - 15.2-era L2 IP Base image ("high iron" build).
  - Lighter feature set; good for basic VLAN/trunk labs.

**How to use:** Add a `Cisco IOL` node, select an L2 `.bin`, set Ethernet to
4 or 8, and connect it to the FGT, routers, and PCs. Configure VLANs and
`switchport` commands as on a Cisco switch.

### IOL L3 — lightweight Cisco routers

- **`L3-ADVENTERPRISEK9-M-15.2-M5.3.bin`**
  - 15.2-era L3 advanced-enterprise image.
- **`i86bi_LinuxL3-AdvEnterpriseK9-M2_157_3_May_2018.bin`**
  - IOS 15.7(3)M2 L3 advanced-enterprise image; newer.

**How to use:** Add a `Cisco IOL` node, select an L3 `.bin`, and use it as a
router: OSPF/EIGRP/BGP, NAT, ACLs, DHCP, router-on-a-stick. L3 images are
routers, not switches; use L2 images for switchports.

### `iourc` — IOL license

- **Path:** `/opt/unetlab/addons/iol/bin/iourc`
- **What:** A hostname-bound IOL license file. It is not an image.
- **Source:** Supplied with the IOL image set; in the reference environment it
  matches the EVE hostname `eve-ng`.
- **How to use:** Keep it in `/opt/unetlab/addons/iol/bin/`. If the EVE hostname
  changes, regenerate or replace it. Without it, IOL nodes will not start.
- **Do not commit it.** It is listed in `.gitignore`.

## File-name hints

Cisco image names encode useful information:

- `adventerprisek9` — advanced enterprise feature set with K9 crypto.
- `ipbasek9` — IP Base feature set with K9 crypto.
- `M` — mainline IOS release.
- `T` — technology release.
- `S` — service-provider release.
- `.image` — Dynamips IOS image.
- `.bin` — IOL binary.

When in doubt, start with the advanced-enterprise image for the platform you are
using.
