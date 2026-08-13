# System Configuration Reference

## Overview

OpenWRT system configuration uses UCI stored in `/etc/config/system`.

## Package Management

### Update Package Lists

```bash
opkg update
```

### Install Package

```bash
opkg install <package_name>
```

### Remove Package

```bash
opkg remove <package_name>
```

### List Installed Packages

```bash
opkg list-installed
```

### Search for Package

```bash
opkg find <package_name>
```

## System Settings

### Change Hostname

```bash
uci set system.@system[0].hostname='<new_hostname>'
uci commit system
/etc/init.d/system reload
```

### Set Timezone

```bash
uci set system.@system[0].timezone='<timezone>'
uci set system.@system[0].zonename='<zone_name>'
uci commit system
/etc/init.d/system reload
```

Example timezones:
- `CET-1CEST,M3.5.0,M10.5.0/3` (Europe/Stockholm)
- `UTC0` (UTC)
- `EST5EDT,M3.2.0,M11.1.0` (America/New_York)

## System Logs

### View System Log

```bash
logread
```

### View Real-time Log

```bash
logread -f
```

### Filter Logs by Service

```bash
logread | grep <service_name>
```

## System Information

### Show System Info

```bash
ubus call system board
```

### Check Uptime

```bash
uptime
```

### Check Memory Usage

```bash
free
```

### Check Disk Usage

```bash
df -h
```

## Backup and Restore

### Create Backup

```bash
sysupgrade --create-backup /tmp/backup-$(date +%Y%m%d).tar.gz
```

### Restore Backup

```bash
sysupgrade --restore-backup /tmp/backup.tar.gz
```

## Verification

Show system configuration:
```bash
uci show system
```

Check system status:
```bash
/etc/init.d/system status
```
