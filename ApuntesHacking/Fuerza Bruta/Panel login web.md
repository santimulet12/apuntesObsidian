Primero, interceptaremos una petición de login con Burpsuite para poder ver de qué manera se envian los datos:

![[Pasted image 20250715183611.png]]

![[Pasted image 20250715184227.png]]

Ahora con la herramienta `hydra` vamos a proceder a hacer fuerza bruta:

```
hydra -t 64 -l admin -P /usr/share/SecLists/rockyou.txt 172.17.0.3 http-post-form "/login.php:username=a&password=^PASS^:Credenciales incorrectas."
```

 Desglose del comando:

**`hydra`** - Es la herramienta principal para ataques de fuerza bruta contra servicios de autenticación.
**`-t 64`** - Especifica el número de hilos (threads) concurrentes. En este caso, 64 conexiones simultáneas para acelerar el ataque.
**`-l admin`** - Define el nombre de usuario específico a probar. La "l" minúscula indica un usuario único (en este caso "admin").
**`-P /usr/share/SecLists/rockyou.txt`** - Especifica el archivo de diccionario de contraseñas. La "P" mayúscula indica una lista de contraseñas. RockYou es un diccionario muy común con millones de contraseñas reales filtradas.
**`172.17.0.3`** - Es la dirección IP del objetivo (típicamente un contenedor Docker por el rango de IP).
**`http-post-form`** - Especifica el módulo de Hydra para ataques contra formularios web que usan método POST.
**`"/login.php:username=a&password=^PASS^:Credenciales incorrectas."`** - Esta es la configuración del formulario POST con tres partes separadas por dos puntos:
- `/login.php` - La URL del formulario de login
- `username=a&password=^PASS^` - Los parámetros POST donde `^PASS^` es el placeholder que Hydra reemplaza con cada contraseña del diccionario
- `Credenciales incorrectas.` - El texto que aparece en la respuesta cuando las credenciales son incorrectas (para que Hydra sepa cuándo falló)

![[Pasted image 20250715190623.png]]
