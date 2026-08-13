# Wireless Configuration Reference

## Overview

OpenWRT wireless configuration uses UCI stored in `/etc/config/wireless`.

## WiFi Network Configuration

### Add New WiFi Network (SSID)

```bash
uci set wireless.<radio>.device='<radio_name>'
uci set wireless.<radio>.network='<network_interface>'
uci set wireless.<radio>.mode='ap'
uci set wireless.<radio>.ssid='<ssid_name>'
uci set wireless.<radio>.encryption='<encryption_type>'
uci set wireless.<radio>.key='<password>'
uci commit wireless
wifi reload
```

### Example: Add Guest WiFi

```bash
uci set wireless.guest=wifi-iface
uci set wireless.guest.device='radio0'
uci set wireless.guest.network='guest'
uci set wireless.guest.mode='ap'
uci set wireless.guest.ssid='Guest-Network'
uci set wireless.guest.encryption='psk2'
uci set wireless.guest.key='guestpassword123'
uci commit wireless
wifi reload
```

## Radio Configuration

### Enable/Disable Radio

```bash
# Disable
uci set wireless.<radio>.disabled='1'

# Enable
uci set wireless.<radio>.disabled='0'

uci commit wireless
wifi reload
```

### Change WiFi Channel

```bash
uci set wireless.<radio>.channel='<channel_number>'
uci commit wireless
wifi reload
```

## Security Settings

### Common Encryption Types

- `none` - No encryption (open network)
- `psk` - WPA-PSK (TKIP)
- `psk2` - WPA2-PSK (AES)
- `psk-mixed` - WPA/WPA2-PSK mixed
- `sae` - WPA3-SAE
- `sae-mixed` - WPA2/WPA3 mixed

### Change WiFi Password

```bash
uci set wireless.<iface>.key='<new_password>'
uci commit wireless
wifi reload
```

## Verification

Show wireless configuration:
```bash
uci show wireless
```

Check WiFi status:
```bash
wifi status
```

List available radios:
```bash
uci show wireless | grep radio
```

Scan for nearby networks:
```bash
iwinfo <interface> scan
```
