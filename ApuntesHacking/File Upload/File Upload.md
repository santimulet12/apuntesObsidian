![[Pasted image 20250616173620.png]]

Aquí tenemos un panel de subida de archivos. Intentaremos subir un archivo de texto para probar si es vulnerable.

![[Pasted image 20250616173902.png]]
![[Pasted image 20250616174047.png]]

Ahora buscaremos el directorio donde deberían estar almacenados los archivos que hemos subido.

```
gobuster dir -u http://<IP>/ -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt 
```
![[Pasted image 20250616174510.png]]

Vemos el directorio **/uploads**, allí buscaremos los archivos subidos

![[Pasted image 20250616174708.png]]
Se ha subido.
Ahora Intentaremos subir una reverse shell en php de MonkeyPentester, de la página [Reverse Shell Generator](https://www.revshells.com/) 

![[Pasted image 20250616175328.png]]

Ahora subiremos la reverse shell a la página.

![[Pasted image 20250616175540.png]]

Debemos ponernos en escucha por el puerto que colocamos en el código php.

```
sudo nc -lvnp 443
```
![[Pasted image 20250616175749.png]]

Ahora debemos acceder al recurso que hemos subido.

![[Pasted image 20250616175911.png]]
![[Pasted image 20250616180128.png]]
