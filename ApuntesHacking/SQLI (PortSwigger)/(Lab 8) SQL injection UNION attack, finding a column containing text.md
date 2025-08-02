Objetivo: Hacer que la base de datos recupere la cadena: 'VCCsqY'

![[Pasted image 20250719214741.png]]

Sabemos que haciendo uso de `UNION SELECT` podemos mostrar la cadena que queramos, para ello debemos saber la cantidad de columnas que posee la tabla, haciendo uso de `ORDER BY` o insertando cadenas con en cada una de las columnas con`UNION SELECT` hasta que encontremos la cantidad total de columnas.

![[Pasted image 20250719215124.png]]

Al ver que la tabla posee 3 columnas, podemos intentar mostrar la cadena que queramos, haciendo uso de `UNION SELECT`.
Consulta con UNION SELECT: `' UNION SELECT 'VCCsqY',NULL,NULL-- -`

![[Pasted image 20250719215456.png]]

Es normal que algunas columnas no admitan representar cadenas de texto, por lo que deberíamos ir fuzzeando cual de las culmnas lo admiten, para poder comenzar a filtrar información de la BBDD.