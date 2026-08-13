# Firewall Configuration Reference

## Overview

OpenWRT firewall configuration uses UCI stored in `/etc/config/firewall`.

## Port Forwarding

### Add Port Forward Rule

```bash
uci add firewall redirect
uci set firewall.@redirect[-1].name='<rule_name>'
uci set firewall.@redirect[-1].src='wan'
uci set firewall.@redirect[-1].src_dport='<external_port>'
uci set firewall.@redirect[-1].dest='lan'
uci set firewall.@redirect[-1].dest_ip='<internal_ip>'
uci set firewall.@redirect[-1].dest_port='<internal_port>'
uci set firewall.@redirect[-1].proto='tcp udp'
uci set firewall.@redirect[-1].target='DNAT'
uci commit firewall
/etc/init.d/firewall restart
```

### Example: Forward port 32400 to Plex server

```bash
uci add firewall redirect
uci set firewall.@redirect[-1].name='Plex Server'
uci set firewall.@redirect[-1].src='wan'
uci set firewall.@redirect[-1].src_dport='32400'
uci set firewall.@redirect[-1].dest='lan'
uci set firewall.@redirect[-1].dest_ip='<internal_ip>'
uci set firewall.@redirect[-1].dest_port='32400'
uci set firewall.@redirect[-1].proto='tcp'
uci set firewall.@redirect[-1].target='DNAT'
uci commit firewall
/etc/init.d/firewall restart
```

## Traffic Rules

### Allow Incoming Traffic

```bash
uci add firewall rule
uci set firewall.@rule[-1].name='<rule_name>'
uci set firewall.@rule[-1].src='wan'
uci set firewall.@rule[-1].dest_port='<port>'
uci set firewall.@rule[-1].proto='tcp udp'
uci set firewall.@rule[-1].target='ACCEPT'
uci commit firewall
/etc/init.d/firewall restart
```

## Verification

List all redirect rules:
```bash
uci show firewall | grep redirect
```

List all firewall rules:
```bash
uci show firewall | grep rule
```

Check firewall status:
```bash
/etc/init.d/firewall status
```
