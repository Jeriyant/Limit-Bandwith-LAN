
# LIMIT BANDWITH LAN DENGAN AUTO SIMPLE QUEUE
Cara Membatasi Bandwidth LAN Secara Otomatis Berdasarkan IP.

## Konfigurasi Utama
### 1. Buat Simple Queue Sebagai Induk Utama (Master)
Gunakan perintah berikut untuk membuat limiter utama:
```shell
/queue simple add name=0.MASTER-BANDWIDTH queue=pcq-upload-default/pcq-download-default target=Bridge
```

### 2. Buat Simple Queue untuk LAN/Hotspot
Gunakan perintah berikut untuk membuat limiter untuk LAN atau Hotspot:
```shell
/queue simple add name=3.LIMITER-LAN parent=0.MASTER-BANDWIDTH queue=pcq-upload-default/pcq-download-default target=Bridge
```

## Script Otomatis untuk Menambahkan Queue
### Petunjuk Pengaturan
1. **Modifikasi sesuai kebutuhan**:
   - Pada `find name="DHCP-Server"` sesuaikan dengan nama DHCP Server Anda.
   - Pada `parent="3.LIMITER-LAN"` sesuaikan dengan nama Induk Simple Queue yang Anda buat.
   - Pada konfigurasi `limit-at=1M/1M max-limit=2M/2M burst-limit=6M/10M burst-threshold=2M/2M burst-time=120/120` sesuaikan dengan limit bandwidth yang Anda inginkan.

### Script
Berikut adalah script lengkap untuk menambahkan queue secara otomatis:
```shell
/ip dhcp-server set [find name="DHCP-Server"] lease-script=":if (\$leaseBound = 1) do={
    :local hostname \$\"lease-hostname\"
    :local ip \$\"leaseActIP\"
    :local queuename (\"Queue_\" . \$hostname)

    # Cek apakah hostname kosong, jika kosong gunakan IP
    :if (\$hostname = \"\") do={
        :set queuename (\"Queue_\" . \$ip)
    }

    # Buat Simple Queue dengan Parent \"3.LIMITER-LAN\" dan konfigurasi limit-at serta burst
    /queue simple add name=\$queuename target=\$ip limit-at=1M/1M max-limit=2M/2M burst-limit=6M/10M burst-threshold=2M/2M burst-time=120/120 parent=\"3.LIMITER-LAN\"
} else={
    :local hostname \$\"lease-hostname\"
    :local ip \$\"leaseActIP\"
    :local queuename (\"Queue_\" . \$hostname)

    # Cek apakah hostname kosong, jika kosong gunakan IP
    :if (\$hostname = \"\") do={
        :set queuename (\"Queue_\" . \$ip)
    }

    # Hapus Simple Queue jika perangkat terputus
    /queue simple remove [find name=\$queuename]
}
"
```

## Pengaturan Lease Time
Untuk memastikan queue limit diperbarui dengan cepat, atur waktu sewa (lease-time) DHCP agar tidak terlalu lama:
```shell
/ip dhcp-server set [find name="DHCP-Server"] lease-time=00:01:00
```

## Uji Coba
1. Jika ada perangkat baru yang terhubung, simple queue akan dibuat secara otomatis.
2. Jika perangkat tersebut terputus dari jaringan (1 menit setelah terputus), maka simple queue yang dibuat akan otomatis dihapus.
