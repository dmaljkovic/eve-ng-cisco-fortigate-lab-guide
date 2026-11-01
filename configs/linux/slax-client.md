# Slax client commands

Slax 9.11 is a small Debian-based Linux image. Use the EVE-NG VNC console. Log
in as `root` (the image may not prompt for a password), or prefix commands with
`sudo`.

## Find the interface

```bash
ip link
```

Usually the interface is `eth0`. Bring it up if needed:

```bash
ip link set eth0 up
```

## DHCP

```bash
dhclient eth0
ip addr show eth0
ip route
```

If `dhclient` is unavailable:

```bash
udhcpc -i eth0
```

Release and retry:

```bash
dhclient -r eth0
dhclient eth0
```

## Static address

SOHO lab (PC1 `.101`, PC2 `.102`):

```bash
ip addr flush dev eth0
ip addr add 192.168.10.101/24 dev eth0
ip link set eth0 up
ip route replace default via 192.168.10.1
```

Small-enterprise lab:

```bash
# PC1, VLAN 10
ip addr flush dev eth0
ip addr add 10.10.0.101/24 dev eth0
ip link set eth0 up
ip route replace default via 10.10.0.1

# PC2, VLAN 20
ip addr flush dev eth0
ip addr add 10.20.0.101/24 dev eth0
ip link set eth0 up
ip route replace default via 10.20.0.1

# PC3, VLAN 10
ip addr flush dev eth0
ip addr add 10.10.0.102/24 dev eth0
ip link set eth0 up
ip route replace default via 10.10.0.1

# PC4, VLAN 20
ip addr flush dev eth0
ip addr add 10.20.0.102/24 dev eth0
ip link set eth0 up
ip route replace default via 10.20.0.1
```

## DNS

For a closed lab, test with IP addresses. If you have real Internet access:

```bash
printf 'nameserver 1.1.1.1\nnameserver 8.8.8.8\n' > /etc/resolv.conf
```

## Tests

SOHO:

```bash
ping -c 4 192.168.10.1
ping -c 4 192.168.10.102
ping -c 4 203.0.113.1
ping -c 4 8.8.8.8
```

Small enterprise:

```bash
ping -c 4 10.10.0.1
ping -c 4 10.20.0.101
ping -c 4 10.0.0.1
ping -c 4 8.8.8.8
```

Check the route and ARP table:

```bash
ip route
ip neigh
```

## Notes

- Use `ip`, not `ifconfig`; minimal images may not include legacy tools.
- If the image is a live system, network changes may not survive a reboot.
  Reapply them or automate them in the lab.
- A blank VNC console usually means the PC is still booting; wait and reconnect.
- If DHCP fails, check the switch access VLAN and the FGT/core DHCP pool before
  falling back to a static address.
