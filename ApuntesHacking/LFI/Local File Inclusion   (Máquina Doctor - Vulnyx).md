![[Pasted image 20250704223040.png]]

Presionamos F12 para comenzar a inspeccionar los elementos de la página

![[Pasted image 20250704223435.png]]

Vemos que accede al archivo `Doctors.html` mediante `doctor-item.php?include=` aquí  podemos probar si podemos acceder a recursos internos de la máquina, haciendo uso de path traversal `doctor-item.php?include=../../../../../../etc/passwd`

![[Pasted image 20250704223750.png]]

Vemos el usuario **admin** que tiene su directorio personal en `/home/admin`. Veremos si tiene claves/certificados de acceso por ssh, con la siguiente ruta `doctor-item.php?include=../../../../../../home/admin/.ssh/id_rsa`

![[Pasted image 20250704224059.png]]

Existe. Pero está protegida por contraseña, podemos intentar crackearla y en caso de que la contraseña sea débil podremos averiguarla y acceder por ssh.
