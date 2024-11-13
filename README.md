# Limit-Bandwith-LAN
Cara Limit Bandwith LAN Berdasarkan IP Secara Otomatis

# Buat Simple Queue Sebagai Induk Utama atau Master
## Limiter Induk
```
/queue simple add name=0.MASTER-BANDWITH queue=pcq-upload-default/pcq-download-default target=Bridge
```
## Limiter LAN/Hotspot
```
/queue simple add name=3.LIMITER-LAN parent=0.MASTER-BANDWITH queue=pcq-upload-default/pcq-download-default target=Bridge
```
## Script Otomatis Menambahkan Queue
Silakan Modifikasi Sesuai Kebutuhan Anda, Pada [find name="DHCP-Server"] sesuaikan dengan nama DHCP Server Anda
Pada parent=\"3.LIMITER-LAN\" sesuaikan dengan nama Induk Simple Queue Anda
Pada limit-at=1M/1M max-limit=2M/2M burst-limit=6M/10M burst-threshold=2M/2M burst-time=120/120 Sesuaikan Dengan Limit yang anda Mau
```
/ip dhcp-server set [find name="DHCP-Server"] lease-script=":if (\$leaseBound = 1) do={\r\
    \n    :local hostname \$\"lease-hostname\"\r\
    \n    :local ip \$\"leaseActIP\"\r\
    \n    :local queuename (\"Queue_\" . \$hostname)\r\
    \n    \r\
    \n    # Cek apakah hostname kosong, jika kosong gunakan IP\r\
    \n    :if (\$hostname = \"\") do={\r\
    \n        :set queuename (\"Queue_\" . \$ip)\r\
    \n    }\r\
    \n    \r\
    \n    # Buat Simple Queue dengan Parent \"3.LIMITER-LAN\" dan konfigurasi limit-at serta Burst\r\
    \n    /queue simple add name=\$queuename target=\$ip limit-at=1M/1M max-limit=2M/2M burst-limit=6M/10M burst-threshold=2M/2M burst-time=120/120 parent=\"3.LIMITER-LAN\"\r\
    \n} else={\r\
    \n    :local hostname \$\"lease-hostname\"\r\
    \n    :local ip \$\"leaseActIP\"\r\
    \n    :local queuename (\"Queue_\" . \$hostname)\r\
    \n    \r\
    \n    # Cek apakah hostname kosong, jika kosong gunakan IP\r\
    \n    :if (\$hostname = \"\") do={\r\
    \n        :set queuename (\"Queue_\" . \$ip)\r\
    \n    }\r\
    \n    \r\
    \n    # Hapus Simple Queue jika perangkat terputus\r\
    \n    /queue simple remove [find name=\$queuename]\r\
    \n}\r\
    \n"
```
