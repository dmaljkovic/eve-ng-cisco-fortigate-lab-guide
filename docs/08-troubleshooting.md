# Troubleshooting

Work from the bottom up: EVE-NG VM → image/permissions → node boot → link →
addressing/routing → firewall/NAT.

## EVE-NG UI or VM

### The web UI does not load

On the host:

```bash
virsh list --all
virsh domifaddr eve-ng
```

On the EVE VM (console or SSH):

```bash
ip -brief address
systemctl status apache2 --no-pager
systemctl status mysql --no-pager
```

- Confirm the VM is running and has the expected IP.
- Confirm Apache is listening on port 80:
  ```bash
  ss -ltnp | grep ':80'
  ```
- If the IP changed, update the DHCP reservation or use the new address.
- If `virsh` uses the wrong URI, set:
  ```bash
  mkdir -p ~/.config/libvirt
  printf 'uri_default = "qemu:///system"\n' > ~/.config/libvirt/libvirt.conf
  ```

### Image is missing from the dropdown

On the EVE VM:

```bash
ls -la /opt/unetlab/addons/qemu/
ls -la /opt/unetlab/addons/dynamips/
ls -la /opt/unetlab/addons/iol/bin/
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions
```

Check:

- QEMU: the folder name is the image name; the primary disk is
  `virtioa.qcow2` or `hda.qcow2`.
- Dynamips: the file ends in `.image`.
- IOL: the file ends in `.bin`, is executable, and `iourc` is present.
- Permissions are fixed after copying.
- Refresh the browser; if needed, `systemctl reload apache2`.

### Image copied but node will not start

- QEMU: check disk name (`virtioa.qcow2`/`hda.qcow2`), disk format, and free
  space:
  ```bash
  df -h /
  qemu-img info /opt/unetlab/addons/qemu/<image>/virtioa.qcow2
  qemu-img check /opt/unetlab/addons/qemu/<image>/virtioa.qcow2
  ```
- Dynamips: check the `.image` file is complete and readable.
- IOL: check the executable bit and `iourc`.

EVE-NG logs are under `/opt/unetlab/data/Logs/`:

```bash
ls -lt /opt/unetlab/data/Logs/ | head
tail -n 100 /opt/unetlab/data/Logs/<relevant-log>
```

## IOL nodes

### IOL node immediately stops

Most common causes:

1. **`iourc` hostname mismatch.**
   ```bash
   hostname
   ls -l /opt/unetlab/addons/iol/bin/iourc
   ```
   The hostname must match the name in the license file. If you renamed the EVE
   VM or cloned it under a different name, regenerate/replace `iourc`.
2. **Image is not executable.**
   ```bash
   chmod +x /opt/unetlab/addons/iol/bin/*.bin
   /opt/unetlab/wrappers/unl_wrapper -a fixpermissions
   ```
3. **Wrong image type.** L2 images are switches; L3 images are routers. Do not
   expect `switchport` commands on an L3 image.
4. **Not enough Ethernet interfaces.** The IOL template defaults to 1. Set the
   count to 4/8 in the node settings before starting.

### IOL interfaces are missing

Increase the **Ethernet** count in the node settings, then stop/wipe/start the
node. Check with:

```text
show ip interface brief
show interfaces status
```

## Cisco Dynamips routers

### A c3725 module is not present

- Stop the node.
- Set `Slot 1`/`Slot 2` to `NM-1FE-TX` or `NM-16ESW`.
- Wipe the node.
- Start it again.
- Check `show ip interface brief` (routed ports) or `show interfaces status`
  (switchports).

Interface naming:

- `NM-1FE-TX` in slot 1 → `FastEthernet1/0`.
- `NM-16ESW` in slot 1 → `Ethernet1/0`–`Ethernet1/15`.

### c7200 will not boot or is very slow

- Raise RAM to 512 MB or more for IOS 15.2.
- Confirm the NPE is `npe-400` (the template default).
- Wait longer; the first boot can be slow.
- Add only the PA modules you need.

### High CPU usage

- Dynamips uses an `idlepc` value from the template. If CPU is still high, use
  the EVE-NG idlepc calculation for the node.
- Do not run more routers than you need.
- IOL is often lighter than Dynamips for simple routing.

## FortiGate

### Console is blank

- Wait 2–5 minutes; FortiOS boot is slow.
- Confirm the node has 2 GB RAM.
- Reconnect the telnet console.
- Check the EVE log for the node.

### First login

- Default user: `admin`.
- Default password: empty on a fresh image.
- FortiOS prompts you to set a new password. Set a strong one and record it
  outside the lab.

### PC can reach the FGT LAN but not the Internet

Check, in order:

1. FGT WAN link is up and has the correct IP:
   ```text
   get system interface physical
   ```
2. Default route:
   ```text
   get router info routing-table all
   ```
3. LAN-to-WAN policy exists and has NAT enabled:
   ```text
   show firewall policy
   ```
4. ISP router is up and reachable:
   ```text
   execute ping 203.0.113.1
   ```
5. Packet capture on the LAN interface:
   ```text
   diagnose sniffer packet port2 'icmp' 4
   ```
6. DNS: in the closed lab, use IP addresses. For real Internet access, configure
   valid DNS servers.

### FortiGate does not route to the VLANs

In the enterprise lab, the FGT needs static routes to `10.10.0.0/24` and
`10.20.0.0/24` via `10.0.0.2`. Check:

```text
get router info routing-table all
```

## Linux/Slax PCs

### No DHCP address

```bash
ip link
dhclient eth0
# or:
udhcpc -i eth0
ip addr show eth0
ip route
```

Check:

- The correct interface name (`eth0`, `ens3`, etc.).
- The switch access port VLAN.
- The FGT DHCP server (SOHO) or core DHCP pool (enterprise).
- The link is connected in EVE and the switch port is up.

Static fallback:

```bash
ip addr add 192.168.10.101/24 dev eth0
ip link set eth0 up
ip route add default via 192.168.10.1
```

For the enterprise lab, use the matching VLAN subnet and gateway.

### VNC console is blank or disconnected

- Wait for the Linux boot.
- Reopen the console.
- Stop and start the node.
- Wipe the node if the disk state is corrupted (this resets the PC).

## Switching and VLANs

### Trunk is not passing VLANs

On the switch:

```text
show interfaces trunk
show vlan brief
show interfaces switchport
```

Check:

- The uplink is `switchport mode trunk`.
- The allowed VLAN list includes 10 and 20.
- The access ports are in the correct VLAN.
- The core subinterfaces use the matching `encapsulation dot1Q <vlan>`.
- The physical link is up in EVE.

If `switchport trunk encapsulation dot1q` is rejected, omit it; the IOL L2
image uses 802.1Q by default.

### PCs get addresses from the wrong VLAN

- Check the access port VLAN on the switch.
- Check the DHCP pool/gateway on the core.
- Check the trunk allowed VLAN list.

### No inter-VLAN routing

On the core:

```text
show ip route
show ip interface brief
```

- `ip routing` must be enabled.
- The subinterfaces for both VLANs must be up.
- Each PC's default gateway must be the core's VLAN interface.
- The FGT must have routes to both VLAN subnets (enterprise lab).

## Performance and memory

On the EVE VM:

```bash
free -h
top
```

- Lower Slax PCs to 1 GB.
- Keep the FGT at 2 GB.
- Use the c3725 rather than the c7200 for the core.
- Start only the nodes you need.
- If the VM swaps, stop unused nodes or increase VM RAM.

## Reset options

- **Node:** stop → wipe → start.
- **Lab:** remove and rebuild the node links if a link is corrupted.
- **Cisco config:** `write erase` then reload, or wipe the node in EVE.
- **FortiGate:** `execute factoryreset` resets the VM; use only if you intend to
  start over.
- **Image permissions:** rerun
  `/opt/unetlab/wrappers/unl_wrapper -a fixpermissions`.

## Useful log locations

- EVE-NG logs: `/opt/unetlab/data/Logs/`
- Apache logs: `/var/log/apache2/`
- System logs: `journalctl -xe`

Do not paste full logs containing passwords, license keys, or public IPs into a
public issue. This repository is private; still avoid committing secrets.
