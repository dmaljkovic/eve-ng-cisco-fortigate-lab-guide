# Glossary

## EVE-NG terms

- **EVE-NG** — Emulated Virtual Environment Next Generation. The lab platform
  that runs the virtual routers, switches, firewalls, and PCs.
- **Lab** — An EVE-NG topology file (`.unl`) containing nodes and links.
- **Node** — One device in a lab: a router, switch, firewall, or PC.
- **Template** — The EVE-NG device definition that tells EVE how to start a
  node (emulation type, default RAM, NIC model, console type, disk layout).
- **Image** — The vendor software file used by a node: a Cisco IOS image, an IOL
  binary, or a QEMU qcow2 disk.
- **Network / Cloud** — An EVE-NG link type. A Cloud can bridge a lab interface
  to a host interface for outside connectivity. For a self-contained lab, use a
  router as the "ISP" instead.
- **Console** — The terminal used to access a node. Cisco routers/switches and
  FortiGate use a telnet serial console; Linux PCs use VNC.
- **Wipe** — Reset a node's saved state/configuration and start it fresh.
- **`unl_wrapper`** — EVE-NG's backend wrapper. The most common post-install
  action is:
  ```bash
  /opt/unetlab/wrappers/unl_wrapper -a fixpermissions
  ```

## Cisco terms

- **Cisco IOS** — The operating system used by Cisco routers and switches.
- **Dynamips** — An open-source emulator that runs real Cisco IOS router images.
  It is used for the c1710, c3725, and c7200 routers.
- **IOL / IOU** — "IOS on Linux" / "IOS on Unix". Cisco IOS images compiled to
  run natively on Linux. EVE-NG runs them without QEMU. IOL L2 images act as
  switches; IOL L3 images act as routers.
- **`iourc`** — The IOL license file. It is tied to the EVE-NG hostname. Without
  a matching `iourc`, IOL nodes will not start.
- **IdlePC** — A Dynamips setting that reduces CPU usage when a router is idle.
  EVE-NG templates ship with a preset value.
- **NM-16ESW** — A 16-port 10/100 switch network module for the c3725. It lets a
  Dynamips router also provide switch ports.
- **PA-FE-TX / PA-4E / PA-8E** — Port adapters for the c7200: one FastEthernet,
  four Ethernet, or eight Ethernet routed ports.
- **VLAN** — A Layer 2 broadcast domain.
- **Trunk** — A link carrying multiple VLANs, normally using 802.1Q tags.
- **Access port** — A switch port assigned to one VLAN, usually for an end device.
- **SVI** — Switch virtual interface; a Layer 3 interface for a VLAN.
- **Router-on-a-stick** — A router with 802.1Q subinterfaces on one link,
  routing between VLANs.
- **DHCP** — Dynamic Host Configuration Protocol; assigns IP addresses to clients.
- **OSPF** — A dynamic routing protocol. The labs use static routes by default;
  OSPF is an optional extension.
- **NAT** — Network Address Translation. The FortiGate translates LAN addresses
  to its WAN address for outside access.

## Fortinet terms

- **FortiGate (FGT)** — The Fortinet firewall. This is the only Fortinet device
  used in this guide.
- **FortiAnalyzer (FAZ)** — Fortinet's logging and analytics appliance. Not
  required for these labs.
- **FortiManager (FMG)** — Fortinet's central management appliance. Not
  required for these labs.
- **FortiOS** — The FortiGate operating system.
- **Policy** — A FortiGate firewall rule that controls traffic and can enable NAT.
- **VDOM** — Virtual domain. Not used in these labs; the default `root` VDOM is
  sufficient.

## Virtualization terms

- **QEMU** — The emulator used by EVE-NG for FortiGate and Linux nodes.
- **qcow2** — A QEMU copy-on-write disk image. FortiGate and Slax images are
  qcow2 files.
- **KVM** — Linux Kernel-based Virtual Machine. EVE-NG uses KVM for QEMU nodes.
- **VNC** — The console protocol used by the Linux/Slax template.
- **Telnet** — The console protocol used by the Cisco and FortiGate templates in
  these labs (inside the lab only; not for production management).
