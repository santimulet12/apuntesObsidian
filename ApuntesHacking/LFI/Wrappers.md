Supongamos que creemos que se acontece un LFI, una de las cosas que deberíamos probar, son los wrappers:

![[Pasted image 20250715220643.png]]

Lo que hará el Wrapper es permite **leer un archivo y codificar su contenido en Base64 antes de devolverlo**.
Intentaremos codificar el `/etc/passwd` con el wrapper `php://filter/convert.base64-encode/resource=/etc/passwd`. Veremos la siguiente respuesta:

```
php://filter/convert.base64-encode/resource=/etc/passwd
```

![[Pasted image 20250715223230.png]]

Ahora, lo que haremos es copiar el hash en Base64 y lo decodificaremos por consola para poder ver el `/etc/passwd`:

```
echo "<base64_hash>" | base64 -d ;echo
```

![[Pasted image 20250715223548.png]]