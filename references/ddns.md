# DDNS Configuration Reference

## Overview

OpenWRT DDNS configuration uses UCI (Unified Configuration Interface) stored in `/etc/config/ddns`.

## Adding a New DDNS Record

### Standard Pattern for Cloudflare DDNS

```bash
uci set ddns.<service_name>=service
uci set ddns.<service_name>.use_ipv6='0'
uci set ddns.<service_name>.enabled='1'
uci set ddns.<service_name>.lookup_host='<subdomain>.<domain>'
uci set ddns.<service_name>.service_name='cloudflare.com-v4'
uci set ddns.<service_name>.domain='<subdomain>@<domain>'
uci set ddns.<service_name>.username='Bearer'
uci set ddns.<service_name>.password='<cloudflare_api_token>'
uci set ddns.<service_name>.ip_source='interface'
uci set ddns.<service_name>.ip_interface='wan'
uci set ddns.<service_name>.interface='wan'
uci set ddns.<service_name>.dns_server='1.1.1.1'
uci set ddns.<service_name>.use_syslog='2'
uci set ddns.<service_name>.check_unit='minutes'
uci set ddns.<service_name>.force_unit='minutes'
uci set ddns.<service_name>.retry_unit='seconds'
uci set ddns.<service_name>.use_https='1'
uci set ddns.<service_name>.cacert='/etc/ssl/certs'
uci commit ddns
/etc/init.d/ddns restart
```

### Example: Adding plex.fldc.se

```bash
uci set ddns.plex=service
uci set ddns.plex.lookup_host='plex.fldc.se'
uci set ddns.plex.domain='plex@fldc.se'
uci set ddns.plex.service_name='cloudflare.com-v4'
uci set ddns.plex.username='Bearer'
uci set ddns.plex.password='<api_token>'
# ... (other common settings)
uci commit ddns
/etc/init.d/ddns restart
```

## Verification

Check configuration:
```bash
uci show ddns.<service_name>
```

View all DDNS services:
```bash
cat /etc/config/ddns
```

Check DDNS service status:
```bash
/etc/init.d/ddns status
```

Check DDNS logs:
```bash
logread | grep ddns
```
