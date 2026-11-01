# EVE-NG node settings

This page covers the templates, resource settings, interfaces, modules, and
consoles used by the labs.

## Template reference

| Template | Type | Image / folder | Default RAM | CPU | NICs | Console |
|---|---|---|---|---|---|---|
| `Fortinet` | QEMU | `fortinet-FGT-v7.0.9-build0444` | 1 GB | 1 | 4 | telnet |
| `Linux` | QEMU | `linux-slax-9.11.0` | 4 GB | 2 | 1 | VNC |
| `1710` | Dynamips | `c1710-bk9no3r2sy-mz.124-23.image` | 96 MB | 1 | built-in | telnet |
| `3725` | Dynamips | `c3725-adventerprisek9-mz.124-15.T14.image` | 256 MB | 1 | 2 onboard + modules | telnet |
| `7206VXR` | Dynamips | `c7200-adventerprisek9-mz.152-4.M11.image` | 256 MB | 1 | PA modules | telnet |
| `Cisco IOL` | IOL | any `.bin` in `iol/bin` | 1 GB | 1 | 1 default | telnet |

Recommended changes:

- FortiGate: **2 GB** RAM (template default 1 GB).
- Linux/Slax: **1 GB**, **1 CPU** (template default 4 GB/2 CPU).
- c7200 with IOS 15.2: **512 MB or more** (template default 256 MB).
- IOL L2/L3: keep **1 GB**; set Ethernet to **4 or 8**.
- c3725/c1710: template defaults are fine.

## Adding a node

1. Open the lab.
2. Use **Add Node**.
3. Pick the template by name.
4. Pick the image from the Image dropdown.
5. Set name, RAM, CPU, and interface count.
6. Add modules if the template has slots.
7. Click Save/Add.

The Image dropdown lists:

- **QEMU:** folder names in `/opt/unetlab/addons/qemu/`.
- **Dynamips:** `.image` files in `/opt/unetlab/addons/dynamips/`.
- **IOL:** `.bin` files in `/opt/unetlab/addons/iol/bin/`.

If the list is empty or stale, see [08-troubleshooting.md](08-troubleshooting.md).

## Connecting nodes

Use the EVE-NG connect tool (or drag from an interface). The labs use Ethernet
links only.

| Platform | Typical interface names |
|---|---|
| FortiGate | `port1`, `port2`, `port3`, `port4` |
| Linux/Slax | `eth0` |
| c3725 | `FastEthernet0/0`, `FastEthernet0/1` |
| c3725 + `NM-1FE-TX` in slot 1 | `FastEthernet1/0` |
| c3725 + `NM-16ESW` in slot 1 | `Ethernet1/0`–`Ethernet1/15` |
| c7200 | `FastEthernet0/0`, `FastEthernet1/0`, … depending on PA slot |
| IOL | `Ethernet0/0`, `Ethernet0/1`, … |

If an interface name is not what you expect, open the console and run:

```text
show ip interface brief
```

For a switch image:

```text
show interfaces status
```

## Modules

### c3725

The `3725` template has two network-module slots:

- `NM-1FE-TX` — one routed FastEthernet port. In slot 1 it appears as
  `FastEthernet1/0`; in slot 2 as `FastEthernet2/0`.
- `NM-16ESW` — sixteen 10/100 switchports. In slot 1:
  `Ethernet1/0`–`Ethernet1/15`; in slot 2: `Ethernet2/0`–`Ethernet2/15`.

Change modules while the node is stopped, then **wipe** and start it. A running
Dynamips node will not pick up a new module cleanly.

### c7200

The `7206VXR` template has six port-adapter slots:

- `PA-FE-TX` — one FastEthernet port.
- `PA-4E` — four Ethernet ports.
- `PA-8E` — eight Ethernet ports.

These are routed ports, not switchports. The interface names depend on the slot
and adapter; use `show ip interface brief`.

### IOL

The `Cisco IOL` template has an **Ethernet** count field. The default is 1;
set it to 4 or 8 for the labs. For a switch, use an L2 image. For a router, use
an L3 image. Interface names are `Ethernet0/0`, `Ethernet0/1`, and so on.

## Consoles

- **Cisco routers/switches:** telnet console. Click the node and open the
  console in the EVE UI.
- **FortiGate:** telnet serial console. First login is `admin` with no
  password; set a strong password immediately.
- **Linux/Slax:** VNC console. Open the node's VNC window; it is a full Linux
  desktop/console.

The consoles are lab-only. Do not expose EVE-NG or these telnet consoles to an
untrusted network.

## Start, stop, and wipe

- **Start:** boots the node and applies its saved state.
- **Stop:** powers it off.
- **Wipe:** resets the node's saved state and configuration. Use it after
  changing modules, changing the image, or breaking a configuration beyond
  easy repair.
- Save the EVE lab after changing the topology.

Cisco running configurations are lost on wipe unless you saved them in the lab's
startup configuration or copied them out. FortiGate and Linux changes live in
the node's disk image until a wipe.

## Image installation and permissions

On the EVE VM:

```bash
# QEMU: copy the image folder
scp -r fortinet-FGT-v7.0.9-build0444 root@192.168.122.40:/opt/unetlab/addons/qemu/

# Dynamips: copy the .image file
scp c3725-adventerprisek9-mz.124-15.T14.image root@192.168.122.40:/opt/unetlab/addons/dynamips/

# IOL: copy the .bin and iourc, then make executables
scp L2-ADVENTERPRISEK9-M-15.2-20150703.bin root@192.168.122.40:/opt/unetlab/addons/iol/bin/
ssh root@192.168.122.40
chmod +x /opt/unetlab/addons/iol/bin/*.bin

# Always finish with:
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions
```

Then refresh the EVE UI. If needed:

```bash
systemctl reload apache2
```

Do not commit vendor images, the `iourc` file, or lab passwords to git.

## IOL license check

On the EVE VM:

```bash
hostname
ls -l /opt/unetlab/addons/iol/bin/iourc
```

The hostname must match the name inside `iourc`. If you rename the EVE VM, IOL
nodes will fail until the license is regenerated or replaced. The license file is
hostname-bound and should stay private.

## Internet access: ISP router vs Cloud

The labs use a c3725 as an ISP stand-in. This is self-contained and repeatable.

To use real Internet access:

1. Add an EVE-NG **Cloud** node.
2. Bridge it to a host interface with NAT (for example, a libvirt/NAT bridge).
3. Connect the Cloud to the FortiGate WAN.
4. Set the FortiGate WAN to DHCP or a static address on that network.
5. Configure DNS and keep the LAN-to-WAN NAT policy.

Be careful: bridging a Cloud to the wrong host interface can expose the lab to
your real LAN. Prefer a dedicated NAT bridge or the closed-lab ISP router.

## Sizing and memory

Recommended totals:

- **SOHO:** about 5.3 GB.
- **Small enterprise:** about 8.5 GB.

The EVE VM has 16 GiB. Avoid starting every node at its template default:

- Lower each Slax PC to 1 GB.
- Set the FortiGate to 2 GB.
- Use the c3725 (256 MB) rather than the c7200 for the enterprise core.
- Start only the nodes you need; you can leave unused PCs powered off.

Check memory on the EVE VM:

```bash
free -h
```

If the VM starts swapping, lower node RAM or stop unused nodes.

## Node naming

Suggested names:

- `ISP`, `FGT`, `CORE`, `SW1`, `SW2`, `PC1`–`PC4`.
- Use descriptions in EVE for links, for example `FGT-WAN`, `TRUNK-VLAN10-20`.

Clear names make the lab easier to troubleshoot and easier to export as
documentation later.
