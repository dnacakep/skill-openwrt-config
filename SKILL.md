---
name: openwrt-config
description: >-
  Panduan pengelolaan router OpenWrt milik Luthfi — "Rumahkiringan" (Amlogic
  S905X/HG680P TV box, OpenWrt 24.10.0 armsr/armv8, hostname Rumahkiringan,
  SSH root@192.168.1.1 via key). Gunakan saat perlu: konfigurasi network &
  firewall (uci), OpenClash proxy (tun fake-ip utun, port 7890-7895), Tailscale,
  Luci & aplikasi PHP di /www (RakitanManager, TinyFM, vnstati), layanan modem
  (sms_tool, ModemManager), monitoring (netdata, nlbwmon, vnstat, smartd),
  cron & script custom (hgled LED, vnstati.sh), atau troubleshooting router.
  Referensi pola UCI generik ada di folder references/.
---

# OpenWrt Rumahkiringan — Amlogic TV Box Router

Router utama "Rumahkiringan": **Android TV box (Amlogic S905X / HG680P-style,
board P212)** yang di-flash OpenWrt 24.10.0. Bukan router WiFi — **ethernet
saja** (br-lan di eth0). Menjadi pusat internet & proxy rumah/skala kecil.

## Kapan skill ini dipakai

- Konfigurasi network/firewall/DHCP lewat UCI (SSH root@192.168.1.1)
- Kelola OpenClash (proxy tun), Tailscale, Luci, aplikasi PHP di /www
- Kelola layanan modem/monitoring: sms_tool, netdata, nlbwmon, vnstat, smartd
- Cek/memodifikasi script custom: `hgled`, `smsled-init.sh`, `vnstati.sh`, RakitanManager
- Rebuild/replikasi router ini ke box Amlogic lain
- Troubleshooting: internet mati, proxy mati, LED indikator, disk eMMC

## Identitas perangkat

| Item | Nilai |
|---|---|
| Hostname | `Rumahkiringan` |
| SoC / board | Amlogic S905X (Meson GXL), board `amlogic,p212` (HG680P-style TV box) |
| OpenWrt | 24.10.0 `r28427-6df0e3d02a`, target `armsr/armv8`, arch `aarch64_generic` |
| RAM | 2 GB (swap 0) |
| Disk | eMMC `mmcblk2`: p1 `/boot` 157M (62%), p2 `/` 960M (44%), p3 960M, p4 4.4G (kosong) |
| Zona waktu | Asia/Jakarta (WIB-7), NTP pool id |
| Akses | SSH dropbear `192.168.1.1:22` (key), Luci `http://192.168.1.1` / `https://192.168.1.1:443` |
| Firewall | fw4 (nftables) |

## Arsitektur jaringan

```
internet (SIM GSM)
   │
   ▼
hilink modem (192.168.8.1)      ← modem "j1"/E3276, WAN OpenWrt
   │  eth1 = 192.168.8.100/24 (DHCP client, DNS 1.1.1.1)
   ▼
OpenWrt Rumahkiringan
   ├─ br-lan (eth0) = 192.168.1.1/24   → LAN (DHCP .100-.248, 12h)
   ├─ utun 198.18.0.1/30               → TUN OpenClash (fake-ip)
   └─ tailscale0 100.115.150.54        → Tailscale (tailnet dnacakep@)
```

- `wan` = eth1 → **hilink modem** (`192.168.8.1`) sebagai default gateway.
- `lan` = br-lan (eth0) → DHCP server, force, 192.168.1.100–248.
- **DNS**: dnsmasq `noresolv 1` + `server 127.0.0.1#7874` → semua DNS lewat
  OpenClash (fake-ip 198.18.0.1/16). DNS listen di 192.168.1.1, 192.168.8.100,
  198.18.0.1, dan 100.115.150.54 (Tailscale).
- Static host DHCP: `armbian` → 192.168.1.192 (MAC 00:E0:4C:68:14:DD).
- IPv6 off (tidak ada wan6).

## UCI & file config penting

| Config | Isi |
|---|---|
| `/etc/config/network` | br-lan (eth0), lan 192.168.1.1/24, wan dhcp eth1 + dns 1.1.1.1 |
| `/etc/config/dhcp` | dnsmasq (resolv ke 127.0.0.1#7874), DHCP lan, host static armbian |
| `/etc/config/firewall` | default ACCEPT lan, REJECT+masq wan, include `nikki` & `openclash` |
| `/etc/config/openclash` | proxy port 7890/7891/7892/7893/7895, dns 7874, cn 9090, mode rule, geo update |
| `/etc/config/nikki` | `enabled '0'` (dinonaktifkan; OpenClash yang dipakai) |
| `/etc/config/tailscale` | enabled, port 41641, fw nftables, acceptDNS, access tsfwlan/tsfwwan/lanfwts |
| `/etc/config/uhttpd` | /www, .php → php-cgi, Luci, https 443 (cert self-signed) |
| `/etc/config/rakitanmanager` | enabled=1, branch dev; telegram (token — JANGAN disebar) |
| `/etc/config/sms_tool` | at/read port `/dev/ttyAML0`, prefix 48 (ID), storage SM, LED |
| `/etc/config/dropbear` | SSH 192.168.1.1:22, PasswordAuth on |
| `/etc/config/system` | hostname, timezone Asia/Jakarta |
| `/etc/crontabs/root` | drop_caches tiap jam; vnstati.sh tiap 5 menit; update geoip/geosite OpenClash tiap Senin |

> **DDNS tidak terpasang** di router ini (tidak ada paket ddns/Cloudflare).
> Pola DDNS ada di `references/ddns.md` jika suatu saat dibutuhkan.

## Layanan yang berjalan

| Layanan | Port | Keterangan |
|---|---|---|
| uhttpd (Luci) | 80/443 | Web admin, theme material |
| OpenClash (clash/mihomo) | 7890 http, 7891 socks, 7892 redir, 7893 mixed, 7895 tproxy, 7874 dns, 9090 controller | Proxy tun utun (fake-ip), config `Starakun.yaml` |
| Nikki | — | Terpasang tapi disabled (`enabled 0`) |
| Tailscale | 41641 | tailnet `dnacakep@`, IP 100.115.150.54 |
| ttyd | 7681 | Web terminal `/bin/bash --login` |
| netdata | 19999 (8125 local) | Monitoring (halaman `/netdata.html`) |
| php8-fpm | 1026 | Untuk aplikasi PHP tambahan |
| dnsmasq | 53 | DNS/DHCP |
| dropbear | 22 | SSH (192.168.1.1) |
| adb | 5037 (127.0.0.1) | adb server di router |
| vnstatd / nlbwmon | — | Monitoring bandwidth (nlbwmon interval 24h, dir /etc/nlbwmon) |
| smartd | — | Monitoring disk eMMC |
| ModemManager / mmcli | — | Terpasang, saat ini tidak ada modem USB terdeteksi (`mmcli -L` kosong) |
| quickstart | 3038 (127.0.0.1) | OpenWrt quickstart |
| hgled (screen session) | — | Indikator internet via LED box (script `/usr/sbin/hgled`) |
| smsled | — | Notifikasi SMS via LED (`/sbin/smsled-init.sh`, paket sms_tool) |

## Aplikasi web di `/www` (PHP via uhttpd + php-cgi)

| App | Path | Fungsi |
|---|---|---|
| **RakitanManager** | `/www/rakitanmanager/` | Manajer monitor modem inject ("IP Hunter"): pantau modem hilink/hp, restart saat mati, notifikasi Telegram |
| **TinyFM** | `/www/tinyfm/` | File manager web (PHP) — akses rootfs via browser |
| **vnstati** | `/www/vnstati/` | Grafik bandwidth (gambar PNG per s/h/d/m/y, digenerate cron tiap 5 menit) |
| Luci | `/cgi-bin/luci` | Admin router |

### RakitanManager (inti monitoring modem)

- Konfigurasi modem: `/usr/share/rakitanmanager/data-modem.json` — daftar `.modems[]`:
  - `jenis: hilink`, `nama: j1`, `portmodem /dev/ttyUSB0`, `iporbit 192.168.8.1`,
    `username/password admin`, `hostbug support.zoom.us`, `modpes modpesv1`,
    `delayping 60`, `status 1`
- Script inti: `core-manager.sh` (loop manajer, baca data-modem.json, kirim Telegram),
  `modem-hilink.sh` (login API hilink SesTokInfo + reconnect/IP hunter),
  `modem-hp.sh`, `modem-mf90.sh`, `modem-orbit.py`, `modem-rakitan.sh`, `plugins/`, `utils.sh`
- UCI: `/etc/config/rakitanmanager` (enabled, branch, token telegram + chatid)
- Log: `/var/log/rakitanmanager.log`
- Alur hilink sama seperti skill `hilink-e3276` (login password_type 4, SesTokInfo).
- **JANGAN** menaruh token Telegram / password modem ke dalam catatan atau skill.

## OpenClash (proxy utama)

- Konfigurasi uci: `/etc/config/openclash`; profil aktif `/etc/openclash/Starakun.yaml`
  (profil lain: `nadiastore.yaml`, `lunatech_simple.yaml`, `lunatech_simple2.yaml`).
- Mode `rule`, `allow-lan true`, `external-controller 0.0.0.0:9090` (secret — jangan disebar),
  TUN: `device utun`, stack system, fake-ip store, sniffer TLS-SNI on.
- Geo data: `/etc/openclash/` (Country.mmdb, GeoIP.dat, GeoSite.dat) — auto-update
  mingguan via cron + uci `geo_custom_url` (rizkikotet-dev GeoSite-WRT, chnr clang.cn).
- Firewall include: `/var/etc/openclash.include` (di-generate OpenClash).
- Kontrol: `uci show openclash`, restart `/etc/init.d/openclash restart`,
  log `/tmp/openclash.log`, watchdog `/usr/share/openclash/openclash_watchdog.sh`.
- DNS router diarahkan ke OpenClash (7874) — jika OpenClash mati, DNS ikut mati.

## Script custom & cron

- `/usr/sbin/hgled` (Internet Indicator, Lutfa Ilham): loop cek `curl bing.com`
  tiap 1 detik → LED box on/off (`hgledon -lan on/off/dis`), jalan via
  `screen -AmdS internet-indicator`. Ctrl: `hgled -r` start, `-s` stop, `-l` loop.
- `/sbin/smsled-init.sh` + `/etc/init.d/smsled` (eko.one.pl / IceG): blink LED saat SMS masuk.
- `/www/vnstati/vnstati.sh`: generate grafik vnstati untuk `br-lan`
  (s/h/d/m/y/t) → cron tiap 5 menit.
- Cron `/etc/crontabs/root`: `vm.drop_caches=3` tiap jam (hemat RAM), update
  geo OpenClash tiap Senin.

## Cara akses & perintah dasar

```bash
SSH="ssh -o StrictHostKeyChecking=no root@192.168.1.1"
$SSH "uci show network; uci show dhcp | head -40"
$SSH "uci set network.lan.ipaddr=192.168.1.2 && uci commit network && /etc/init.d/network reload"
$SSH "logread | tail -50"            # log sistem
$SSH "tail -50 /tmp/openclash.log"   # log proxy
$SSH "tail -50 /var/log/rakitanmanager.log"
$SSH "netstat -tlnp"                 # port yang listen
$SSH "mmcli -L"                      # modem USB (jika ada)
$SSH "tailscale status"
```

## Replikasi ke box Amlogic lain

1. Flash OpenWrt armsr/armv8 (mis. via `luci-app-amlogic` atau Armbian→OpenWrt).
2. `opkg update && opkg install` paket sesuai daftar (openclash, nikki, tailscale,
   ttyd, netdata, smartd, vnstat2/vnstati2, nlbwmon, php8-* , sms-tool, adb, screen, jq).
3. Salin `/etc/config/*` + `/etc/rc.local` + `/etc/crontabs/root`.
4. Salin `/etc/openclash/` (kecuali binary core — sesuaikan arsitektur) + `/etc/nikki/`.
5. Salin `/www/` (rakitanmanager, tinyfm, vnstati) + `/usr/share/rakitanmanager/`.
6. Salin `/usr/sbin/hgled`, `/sbin/smsled-init.sh`, `/etc/init.d/smsled` —
   sesuaikan GPIO/port LED box (`hgledon` binary spesifik box).
7. Set hostname, timezone, SSH key, Tailscale auth (`tailscale up`).
8. Konfigurasi ulang data-modem.json (IP modem, hostbug, delayping).

## Troubleshooting

| Gejala | Cek / Solusi |
|---|---|
| Internet mati total | `ping 8.8.8.8` dari router; cek eth1 (dhcp dapat IP?), modem hilink 192.168.8.1; log RakitanManager (auto-restart modem) |
| DNS error tapi ping IP jalan | OpenClash mati → DNS 7874 ikut mati; restart OpenClash, cek /tmp/openclash.log |
| Proxy lambat/error | Cek profil Starakun.yaml, `clash` proses, tun utun ada? (`ip -br addr`), log OpenClash |
| LED indikator mati padahal internet OK | `hgled -r`; cek `screen -ls`; cek binary `hgledon` ada di box |
| Disk / boot penuh | `df -h`; cek /boot (mmcblk2p1) — hapus kernel/image lama |
| RAM rendah | cron sudah drop_caches; cek proses besar (netdata/php8/clash) |
| Luci lambat | `logread`, cek php-cgi worker, `top` |
| eMMC SMART warning | `smartctl -a /dev/mmcblk2` (paket smartmontools) |
| Tailscale tidak konek | `tailscale status`, `tailscale up` ulang, cek /etc/config/tailscale |

## Referensi pola UCI generik

Folder `references/` berisi pola umum (ddns, firewall, network, system, wireless)
yang tetap berguna untuk penambahan layanan baru — sesuaikan nilai dengan
perangkat di atas.

## File studi (salinan dari router)

- `~/.freebuff/tmp/openwrt-rumahkiringan/etc_config/` — semua file /etc/config
- `etc_openclash/` — Starakun.yaml, nadiastore.yaml, proxy_provider, custom, rule_provider
- `etc_misc/` — crontab, rc.local, nikki firewall include
- `rakitanmanager/` — core-manager.sh, modem-*.sh, plugins, data-modem.json
- `www_rakitan/` `www_tinyfm/` `www_vnstati/` — aplikasi web
- `scripts/` — hgled, smsled-init.sh
