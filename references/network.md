# Network Configuration Reference

## Overview

OpenWRT network configuration uses UCI stored in `/etc/config/network`.

## Static DHCP Leases

### Add Static Lease

```bash
uci add dhcp host
uci set dhcp.@host[-1].name='<hostname>'
uci set dhcp.@host[-1].mac='<mac_address>'
uci set dhcp.@host[-1].ip='<ip_address>'
uci commit dhcp
/etc/init.d/dnsmasq restart
```

### Example: Reserve IP for Plex server

```bash
uci add dhcp host
uci set dhcp.@host[-1].name='plex-server'
uci set dhcp.@host[-1].mac='aa:bb:cc:dd:ee:ff'
uci set dhcp.@host[-1].ip='<lan_ip>'
uci commit dhcp
/etc/init.d/dnsmasq restart
```

## VLAN Configuration

### Create VLAN Interface

```bash
uci set network.<vlan_name>=interface
uci set network.<vlan_name>.proto='static'
uci set network.<vlan_name>.device='<bridge>.<vlan_id>'
uci set network.<vlan_name>.ipaddr='<ip_address>'
uci set network.<vlan_name>.netmask='<netmask>'
uci commit network
/etc/init.d/network restart
```

## Bridge Configuration

### Create Bridge

```bash
uci set network.<bridge_name>=device
uci set network.<bridge_name>.type='bridge'
uci set network.<bridge_name>.ports='<interface1> <interface2>'
uci commit network
/etc/init.d/network restart
```

## DNS Configuration

### Add Custom DNS Entry

```bash
uci add dhcp domain
uci set dhcp.@domain[-1].name='<hostname>'
uci set dhcp.@domain[-1].ip='<ip_address>'
uci commit dhcp
/etc/init.d/dnsmasq restart
```

## Verification

Show network configuration:
```bash
uci show network
```

Show DHCP configuration:
```bash
uci show dhcp
```

Check network status:
```bash
/etc/init.d/network status
```

View active DHCP leases:
```bash
cat /tmp/dhcp.leases
```
