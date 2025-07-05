
**Atacante** --> 10.0.2.4 | **Segmento** --> 10.0.2.0/24
 
Vamos a realizar un reconocimiento de los equipos conectados a la red de la máquina atacante:

```
nmap -sn 10.0.2.0/24
```

o

```
sudo arp-scan -I <Interface> --localnet
```

Interface: eth0, type: EN10MB, MAC: 08:00:27:d1:f8:5d, IPv4: 10.0.2.4
WARNING: Cannot open MAC/Vendor file ieee-oui.txt: Permission denied
WARNING: Cannot open MAC/Vendor file mac-vendor.txt: Permission denied
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
10.0.2.1        52:54:00:12:35:00       (Unknown: locally administered)
10.0.2.2        52:54:00:12:35:00       (Unknown: locally administered)
10.0.2.3        08:00:27:81:10:75       (Unknown)
==10.0.2.5        08:00:27:a8:83:5e       (Unknown)==
==10.0.2.6        08:00:27:48:7a:1f       (Unknown)==
==10.0.2.7        08:00:27:cd:bf:97       (Unknown)==
==10.0.2.9        08:00:27:cc:1d:08       (Unknown)==
==10.0.2.10       08:00:27:30:95:5b       (Unknown)==

![[Pasted image 20250705115302.png]]

![[Pasted image 20250705115336.png]]

