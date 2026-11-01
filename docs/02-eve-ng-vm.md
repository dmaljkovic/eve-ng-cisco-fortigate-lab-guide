# Reference EVE-NG VM

This page documents the reference EVE-NG environment used for the labs. It is
not a complete EVE-NG installation manual; it describes the VM layout, image
paths, and the operations you need for the labs.

## VM specifications

| Item | Value |
|---|---|
| Platform | EVE-NG Community Edition 6.2.0-4 |
| Base OS | Ubuntu 22.04 |
| vCPU | 8 |
| RAM | 16 GiB |
| Disk | 60 GiB qcow2, thin-provisioned |
| CPU model | `host-passthrough` |
| Console | SPICE / QXL |
| Guest kernel | `6.7.5-eveng-6-ksm+` |
| KSM | Enabled |
| Lab network | libvirt default network, `192.168.122.0/24` |
| EVE IP | `192.168.122.40` (DHCP reservation) |
| Web UI | `http://192.168.122.40/` (HTTP, port 80) |

The host provides KVM and nested virtualization. Inside the EVE VM, `/dev/kvm`
must exist for QEMU nodes; IOL and Dynamips do not use QEMU.

## Access

- **Web UI:** `http://192.168.122.40/`
- **Default EVE CE login:** `admin` / `eve` — change this after installation.
- **SSH:** `ssh root@192.168.122.40` — use the password set during installation.
- The EVE web UI is HTTP-only in this reference environment. Do not expose it to
  an untrusted network.

Passwords and license files are intentionally not stored in this repository.

## Host checks

On the libvirt host:

```bash
virsh list --all
virsh domifaddr eve-ng
```

On the EVE VM:

```bash
hostname
ip -brief address
systemctl status apache2 guacd mysql tomcat9 --no-pager
df -h /
free -h
```

If `virsh` uses the wrong libvirt URI, set the default explicitly:

```bash
mkdir -p ~/.config/libvirt
printf 'uri_default = "qemu:///system"\n' > ~/.config/libvirt/libvirt.conf
```

## Image layout on the EVE VM

```text
/opt/unetlab/addons/
├── dynamips/                 Cisco IOS .image files
├── iol/
│   └── bin/                  Cisco IOL .bin files and the iourc license
└── qemu/
    ├── fortinet-FGT-v7.0.9-build0444/
    │   └── virtioa.qcow2
    └── linux-slax-9.11.0/
        └── hda.qcow2
```

Rules:

- The **QEMU folder name** is what appears in the node's Image dropdown.
- The primary QEMU disk is normally `virtioa.qcow2` (virtio) or `hda.qcow2`
  (IDE). Additional disks are `virtiob.qcow2`, `hdb.qcow2`, and so on.
- Dynamips nodes list every `.image` in `/opt/unetlab/addons/dynamips/`.
- IOL nodes list every `.bin` in `/opt/unetlab/addons/iol/bin/`.
- IOL images must be executable, and `iourc` must match the EVE hostname.

## Adding an image

Copy the image into the correct directory, then fix permissions.

QEMU example:

```bash
scp -r fortinet-FGT-v7.0.9-build0444 root@192.168.122.40:/opt/unetlab/addons/qemu/
ssh root@192.168.122.40
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions
```

Dynamips example:

```bash
scp c3725-adventerprisek9-mz.124-15.T14.image root@192.168.122.40:/opt/unetlab/addons/dynamips/
ssh root@192.168.122.40 '/opt/unetlab/wrappers/unl_wrapper -a fixpermissions'
```

IOL example:

```bash
scp L2-ADVENTERPRISEK9-M-15.2-20150703.bin root@192.168.122.40:/opt/unetlab/addons/iol/bin/
ssh root@192.168.122.40
chmod +x /opt/unetlab/addons/iol/bin/*.bin
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions
```

After copying, refresh the EVE UI. If an image does not appear, check the folder
name, the primary disk name, and the permissions. See
[08-troubleshooting.md](08-troubleshooting.md).

## IOL license

`/opt/unetlab/addons/iol/bin/iourc` is a hostname-bound license file. In this
reference environment the EVE hostname is `eve-ng`, and the file matches it.

If you rename the EVE VM or clone it under a different hostname, IOL nodes will
fail to start until the `iourc` file is regenerated or replaced. Do not commit
the license file to git.

## Rebuilding notes

If you rebuild the EVE VM from scratch:

- Use the official EVE-NG CE 6.2.0-4 ISO.
- The ISO's bundled local package repository can be incompatible with an updated
  Ubuntu 22.04 base. The reference build installed EVE-NG from the official
  online repository instead:
  ```text
  deb [trusted=yes] http://www.eve-ng.net/jammy jammy main
  ```
- Use a DHCP-based unattended install for the base OS, then install EVE-NG from
  the online repository.
- Set a non-default admin password and restrict access to the management network.

These notes are for reference; the labs assume EVE-NG is already running.
