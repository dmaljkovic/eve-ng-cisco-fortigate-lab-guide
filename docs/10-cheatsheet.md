# Cheatsheet

## EVE-NG

### VM and UI

```bash
# Host: find the VM and its address
virsh list --all
virsh domifaddr eve-ng

# EVE VM: basic checks
hostname
ip -brief address
systemctl status apache2 --no-pager
df -h /
free -h
```

- UI: `http://192.168.122.40/`
- Default EVE CE login: `admin` / `eve` — change it.
- SSH: `ssh root@192.168.122.40`

### Image paths

```text
/opt/unetlab/addons/dynamips/        Cisco IOS .image files
/opt/unetlab/addons/iol/bin/         Cisco IOL .bin files + iourc
/opt/unetlab/addons/qemu/            QEMU image folders
/opt/unetlab/templates/intel/        EVE templates
/opt/unetlab/labs/                   Lab files
/opt/unetlab/data/Logs/              EVE logs
```

### Permissions and refresh

```bash
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions
systemctl reload apache2
```

### IOL license

```bash
hostname
ls -l /opt/unetlab/addons/iol/bin/iourc
chmod +x /opt/unetlab/addons/iol/bin/*.bin
```

The hostname must match the name inside `iourc`.

## Templates and images

| Template | Image | RAM |
|---|---|---|
| `Fortinet` | `fortinet-FGT-v7.0.9-build0444` | 2 GB |
| `Linux` | `linux-slax-9.11.0` | 1 GB |
| `1710` | `c1710-bk9no3r2sy-mz.124-23.image` | 96 MB |
| `3725` | `c3725-adventerprisek9-mz.124-15.T14.image` | 256 MB |
| `7206VXR` | `c7200-adventerprisek9-mz.152-4.M11.image` | 512 MB+ |
| `Cisco IOL` | any L2/L3 `.bin` | 1 GB |

## Cisco IOS

### General

```text
show ip interface brief
show ip route
show running-config
write memory
```

### Routing and DHCP

```text
show ip route
show ip protocols
show ip dhcp binding
show ip dhcp pool
```

### Switching (IOL L2)

```text
show vlan brief
show interfaces trunk
show interfaces status
show interfaces switchport
show spanning-tree
```

### c3725 modules

```text
show ip interface brief
show interfaces status
```

- `NM-1FE-TX` in slot 1 → `FastEthernet1/0`
- `NM-16ESW` in slot 1 → `Ethernet1/0`–`Ethernet1/15`

### c7200 port adapters

```text
show ip interface brief
show diag
```

- `PA-FE-TX` — one FastEthernet
- `PA-4E` — four Ethernet
- `PA-8E` — eight Ethernet

## FortiGate

### Verify

```text
get system interface physical
get router info routing-table all
get system arp
show system dhcp server
show firewall policy
```

### Test

```text
execute ping 203.0.113.1
execute ping 8.8.8.8
diagnose sniffer packet port2 'icmp' 4
```

### Common config blocks

```text
config system interface
    edit "port1"
        set mode static
        set ip 203.0.113.2 255.255.255.252
        set allowaccess ping
    next
    edit "port2"
        set mode static
        set ip 192.168.10.1 255.255.255.0
        set allowaccess ping https ssh
    next
end

config router static
    edit 1
        set gateway 203.0.113.1
        set device "port1"
    next
end

config firewall policy
    edit 1
        set name "LAN-to-WAN"
        set srcintf "port2"
        set dstintf "port1"
        set action accept
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service "ALL"
        set nat enable
    next
end
```

## Linux/Slax

```bash
ip link
dhclient eth0
# or:
udhcpc -i eth0
ip addr show eth0
ip route
ping -c 4 192.168.10.1
ping -c 4 8.8.8.8
```

Static fallback:

```bash
ip addr add 192.168.10.101/24 dev eth0
ip link set eth0 up
ip route add default via 192.168.10.1
```

## Addressing

### SOHO

| Segment | Network | Addresses |
|---|---|---|
| WAN | `203.0.113.0/30` | ISP `.1`, FGT `.2` |
| ISP loopback | `8.8.8.8/32` | ISP |
| LAN | `192.168.10.0/24` | FGT `.1`, DHCP `.100`–`.199` |

### Small enterprise

| Segment | Network | Addresses |
|---|---|---|
| WAN | `203.0.113.0/30` | ISP `.1`, FGT `.2` |
| FGT–CORE | `10.0.0.0/30` | FGT `.1`, CORE `.2` |
| VLAN 10 | `10.10.0.0/24` | CORE `.1`, DHCP `.100`–`.199` |
| VLAN 20 | `10.20.0.0/24` | CORE `.1`, DHCP `.100`–`.199` |

## Common failures

| Symptom | First checks |
|---|---|
| Image missing | folder/disk name, permissions, refresh |
| IOL stops | `iourc` hostname, executable bit |
| FGT console blank | wait, 2 GB RAM, reconnect |
| PC no DHCP | link, access VLAN, DHCP pool |
| No inter-VLAN | `ip routing`, subinterfaces, gateways |
| No Internet | FGT default route, NAT policy, ISP link |
| c3725 port missing | module, wipe, restart |
| c7200 slow | raise RAM, wait, fewer modules |
