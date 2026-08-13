# Skill: OpenWrt Config — Router "Rumahkiringan" (Amlogic TV Box)

Panduan pengelolaan router OpenWrt milik Muhammad Luthfianto — **"Rumahkiringan"**: Android TV box **Amlogic S905X (HG680P-style)** yang di-flash OpenWrt 24.10.0 (armsr/armv8, 2 GB RAM, eMMC). SSH `root@192.168.1.1` via key. Menjadi pusat internet & proxy dengan WAN dari modem hilink.

## Instalasi

```bash
npx skills add dnacakep/skill-openwrt-config
```

## Kemampuan

- **Konfigurasi network & firewall (UCI)** — br-lan, WAN eth1 → modem hilink, DHCP, DNS via OpenClash
- **OpenClash** — proxy tun fake-ip (utun 198.18.0.1), port 7890–7895, DNS 7874
- **Tailscale** — akses remote aman via tailnet pribadi
- **Aplikasi PHP di /www** — RakitanManager (monitor modem inject + Telegram), TinyFM, vnstati
- **Layanan modem** — sms_tool (SMS via LED), ModemManager
- **Monitoring** — netdata, nlbwmon, vnstat, smartd
- **Cron & script custom** — hgled (indikator internet via LED box), vnstati.sh, drop_caches
- **Replikasi ke box Amlogic lain** + **troubleshooting**

## Isi repo

```
skill-openwrt-config/
├── SKILL.md               ← panduan utama (spesifik perangkat + cara kelola)
├── README.md              ← halaman ini
└── references/
    ├── ddns.md            ← pola DDNS (Cloudflare) generik
    ├── firewall.md        ← pola firewall/port forwarding
    ├── network.md         ← pola DHCP/VLAN/bridge/DNS
    ├── system.md          ← paket & pengaturan sistem
    └── wireless.md        ← pola wireless (untuk router ber-WiFi)
```

## Kapan dipakai

Saat perlu: konfigurasi network/firewall/DHCP lewat UCI, kelola OpenClash / Tailscale / Luci / aplikasi PHP di /www, kelola layanan modem & monitoring, memodifikasi script custom (hgled, sms_tool, vnstati), rebuild router ke box Amlogic lain, atau troubleshooting router (internet mati, DNS error, proxy lambat, disk penuh).

## Lisensi

MIT — lihat [LICENSE](LICENSE).
