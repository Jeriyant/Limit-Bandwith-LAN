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
tes
